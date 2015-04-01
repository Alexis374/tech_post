###django 模板语言：程序员部分
从技术的观点来介绍django模板语言——如何工作以及怎么扩展

####基础
---
`template`是个用django模板语言标记的文本文档，或平常的python字符串，可以包含块标签（`block tags`）和变量
block tags是在template里的标记，用于某些功能，如输出内容，组成控制结构，从数据库取出数据或访问其他模板标签。
它们被包括在`{% %}`之中。
`varible`是输出值的符号，包含在`{{}}`中。
`context`被传递到模板，完成变量名到变量值映射过程。
模板通过 从context里取出变量值替换变量的“洞”和执行所有的块标签，来渲染（`render`）一个上下文(context)

---
####用模板系统
`class Template`
2步：
+ 首先，编译原始模板代码到`Template`对象
+ 调用`Template`对象的`render()`方法，用给定的context填充

#####编译一个字符串
直接调用：
```python
>> from django.template import Template
>>> t = Template("My name is {{ my_name }}.")
>>> print(t)
<django.template.Template instance>
```

>编译的原理：
由于性能原因，系统只解析一次原始模板代码——在建立Template对象的时候，之后，内部存储为“node”结构。
即使解析也是很快的，大多数是通过调用简单的、短的正则表达式完成解析的。

#####渲染上下文
######render(context)
一旦拥有了Template对象，就可以用它渲染一个或多个上下文。Context类存在于[django.template.Context](https://docs.djangoproject.com/en/1.7/ref/templates/api/#django.template.Context)
其构造函数接收2个参数，一个字典，映射着变量名和值；另一个是当前的app名，用于帮助[反向解析命名的url](https://docs.djangoproject.com/en/1.7/topics/http/urls/#topics-http-reversing-url-namespaces),这个参数可以省略
```python
>>> from django.template import Context, Template
>>> t = Template("My name is {{ my_name }}.")

>>> c = Context({"my_name": "Adrian"})
>>> t.render(c)
"My name is Adrian."

>>> c = Context({"my_name": "Dolores"})
>>> t.render(c)
"My name is Dolores."
```
######变量和查询
变量名由字母数字下划线和点组成，点有着特殊含义——查询。
foo.bar 查询顺序：
+字典查询 `foo["bar"]`
+属性查询 `foo.bar`
+列表索引 `foo[bar]`
+方法调用
在模板里{{foo.bar}}被解释为字符串，而不是bar的值，如果context里有bar的话
例子：
```python
>>> from django.template import Context, Template
>>> t = Template("My name is {{ person.first_name }}.")
>>> d = {"person": {"first_name": "Joe", "last_name": "Johnson"}}
>>> t.render(Context(d))
"My name is Joe."

>>> class PersonClass: pass
>>> p = PersonClass()
>>> p.first_name = "Ron"
>>> p.last_name = "Nasty"
>>> t.render(Context({"person": p}))
"My name is Ron."

>>> t = Template("The first stooge in the list is {{ stooges.0 }}.")
>>> c = Context({"stooges": ["Larry", "Curly", "Moe"]})
>>> t.render(c)
"The first stooge in the list is Larry."

>>> class PersonClass2:
...     def name(self):
...         return "Samantha"
>>> t = Template("My name is {{ person.name }}.")
>>> t.render(Context({"person": PersonClass2}))
"My name is Samantha."
```
方法调用会比较复杂，需要记住：
+ 如果在被调用的时候，出现异常，异常会传播，除非异常类有`silent_variable_failure`，并且值为True，这个变量会渲染为` TEMPLATE_STRING_IF_INVALID `设置项指定的字符串值（默认为空值），如：

```python
>>> t = Template("My name is {{ person.first_name }}.")
>>> class PersonClass3:
...     def first_name(self):
...         raise AssertionError("foo")
>>> p = PersonClass3()
>>> t.render(Context({"person": p}))
Traceback (most recent call last):
...
AssertionError: foo

>>> class SilentAssertionError(Exception):
...     silent_variable_failure = True
>>> class PersonClass4:
...     def first_name(self):
...         raise SilentAssertionError
>>> p = PersonClass4()
>>> t.render(Context({"person": p}))
"My name is ."
```
注意 `django.core.exceptions.ObjectDoesNotExist`异常类就有`silent_variable_failure = True`的定义，这个类是所有models.Models类`DoesNotExist`异常的父类，所以在使用django model对象时，所有不存在异常都会静默处理
+ 方法调用中函数必须是没有必须参数的，否则会使用`TEMPLATE_STRING_IF_INVALID`的值。
+ 显然一些函数的调用会有副作用，会造成安全隐患，如model对象的`delete()`方法。为了防止这样做，给callable()的变量增加一个`alters_data=True`，这样django就会使用`TEMPLATE_STRING_IF_INVALID`的值。所有动态生成的`delete() save()`方法无条件自动增加了`alters_data=True`，如：
```python
def sensitive_function(self):
    self.database_record.delete()
sensitive_function.alters_data = True
```
+ 你偶尔可能不需要调用callable的变量，想关闭这个特性。设置属性`do_not_call_in_templates = True`，模板系统会把变量当成不可调用的，这样就可以访问这个变量的属性了 （评注：估计一般没人会这么干）
---
######非法变量如何处理：
若变量不存在，插入 ` TEMPLATE_STRING_IF_INVALID`设置的值
在过滤器上应用非法值时：若` TEMPLATE_STRING_IF_INVALID`为空，过滤器会被应用，否则过滤器被忽略。
在 if for regroup模板标签上时，会有一些不同：若非法值提供给这三个标签，变量被解释为为None，在这些标签里的过滤器会一直被应用为非法的值。
如果` TEMPLATE_STRING_IF_INVALID`值里面含有一个`%s`，这个格式化标记会被替换为非法变量的名称。

**注意** 尽管`TEMPLATE_STRING_IF_INVALID `可以作为很好的调试工具，在开发过程中进行设置也不是一个明智的决定。许多模板，包括admin的，依赖于对不存在的变量进行静默处理，如果这个值被改写，你会遇到渲染的问题。这个值应该只在调试特定模板的时候设置，一旦调试结束需要清除。
######内建变量
每个context都带有 True False None，对应python的变量

---
######字符串的限制
模板语言没有办法转码自己语法的字符。如若需要输出 {% 和%}，需要`templatetag`这个标签，[详见](https://docs.djangoproject.com/en/1.7/ref/templates/builtins/#std:templatetag-templatetag).
同样的问题也存在于你想要在 模板过滤器(template filter)或标签参数(tag arguments)中。例如，当解析block时，parser寻找{%之后第一个出现的 %}，这就要避免"%}"出现在字符串里。下面例子中会出现TemplateSyntaxError
```html
{% include "template.html" tvar="Some string literal with %} in it." %}

{% with tvar="Some string literal with %} in it." %}{% endwith %}
```
同样的问题在过滤器参数中出现时也被触发：
`{{ some.variable|default:"}}" }}`
如果你必须这么做，把他们存放在模板变量里，或用自定义的模板标签或过滤器

----
####与上下文对象打交道
class Context
大多数时候，通过传递一个完全填充的字典来实例化一个Context对象，但你可以在实例化之后增加和删除条目，用字典的方法
```python
>>> from django.template import Context
>>> c = Context({"foo": "bar"})
>>> c['foo']
'bar'
>>> del c['foo']
>>> c['foo']
''
>>> c['newvariable'] = 'hello'
>>> c['newvariable']
'hello'
```
Context 是一个dict的栈。push方法增加一个层级，pop方法弹出一个层级。增加元素只在栈顶的字典上操作。
```python
>>> c = Context()
>>> c['foo'] = 'first level'#c=[{'foo':'first level'}]
>>> c.push()
{}  #c=[{'foo':'first level'},{}]
>>> c['foo'] = 'second level'#c=[{'foo':'first level'},{'foo':'second level'}]
>>> c['foo']
'second level'
>>> c.pop()
{'foo': 'second level'} #c=[{'foo':'first level'}]
>>> c['foo']
'first level'
>>> c['foo'] = 'overwritten'
>>> c['foo']  #c=[{'foo':'overwritten'}]
'overwritten'
>>> c.pop()
Traceback (most recent call last):
...
ContextPopException
```
django 1.7中，push()可以作为上下问管理器，保证pop()方法一定被调用：
```python
>>> c = Context()
>>> c['foo'] = 'first level'
>>> with c.push():
>>>     c['foo'] = 'second level'
>>>     c['foo']
'second level'
>>> c['foo']
'first level'
```
push接收关键字参数，来创建新层级的dict
```python
>>> c = Context()
>>> c['foo'] = 'first level'
>>> with c.push(foo='second level'):
>>>     c['foo']
'second level'
>>> c['foo']
'first level'
```
update方法，接收新字典参数，达到与push同样的效果
```python
>>> c = Context()
>>> c['foo'] = 'first level'
>>> c.update({'foo': 'updated'})
{'foo': 'updated'}
>>> c['foo']
'updated'
>>> c.pop()
{'foo': 'updated'}
>>> c['foo']
'first level'
```

flatten()方法，顾名思义，返回字典的平级化，将字典栈上的所有字典当做一个字典，包括内建的变量.
flatten()是django 1.7新增方法，注意是返回字典的平级化，本身context的值并没有变。
```python
>>> c = Context()
>>> c['foo'] = 'first level'
>>> c.update({'bar': 'second level'})  #c=[{'False': False, 'None': None, 'foo': 'first level', 'True': True}, {'bar': 'second level'}]
{'bar': 'second level'}
>>> c.flatten()
{'True': True, 'None': None, 'foo': 'first level', 'False': False, 'bar': 'second level'}
#c还是与之前的一样。
```
flatten()的结果可以用于单元测试，与字典比较：
```python
class ContextTest(unittest.TestCase):
    def test_against_dictionary(self):
        c1 = Context()
        c1['update'] = 'value'
        self.assertEqual(c1.flatten(), {
            'True': True, 'None': None, 'False': False,
            'update': 'value'})
```

---
#####派生Context类:RequestContext
RequestContext不同之处：
构造时接收 HttpRequest 作为第一个参数，`c = RequestContext(request, {'foo': 'bar',})`
第二个不同是，自动填入了一些变量，这些变量是来自TEMPLATE_CONTEXT_PROCESSORS设置项。
TEMPLATE_CONTEXT_PROCESSORS是一组字符串，每个字符串代表一个可被调用的 上下文处理器(context processors),它们接收request对象，返回一个字典，这个字典被合并到上下文中。默认的TEMPLATE_CONTEXT_PROCESSORS是：
```python
("django.contrib.auth.context_processors.auth",
"django.core.context_processors.debug",
"django.core.context_processors.i18n",
"django.core.context_processors.media",
"django.core.context_processors.static",
"django.core.context_processors.tz",
"django.contrib.messages.context_processors.messages")
```
默认还有`django.core.context_processors.csrf`，由于安全原因，这个被硬编码了，不能关闭。
每个处理器按顺序被应用，即后面出现的同名key的value可能覆盖前面key的value。

上下文处理器(context processors)何时被处理？在context被处理之后。 意味着上下文处理器的变量可能覆盖你自己提供的。所以要注意你自己提供的变量名（即字典的key）
同样，你可以给你自己的RequestContext 对象提供额外的上下文处理器。如果要这样做，增加一个额外的列表作为第三个参数。
```python
from django.http import HttpResponse
from django.template import RequestContext
#context processor 接收request对象，返回一个字典
def ip_address_processor(request):
    return {'ip_address': request.META['REMOTE_ADDR']}
#RequestContext，接收request，同样接收额外的上下文处理器list作为第三个参数
def some_view(request):
    # ...
    c = RequestContext(request, {
        'foo': 'bar',
    }, [ip_address_processor])
    return HttpResponse(t.render(c))
```
注意：用`render_to_response`函数，默认传的是Context而非RequestContext，若要改变，有两种方法：
1. 用`render`代替`render_to_response` 
2. `render_to_response('my_template.html',my_data_dictionary,context_instance=RequestContext(request))`,用context_instance强制。

默认上下文处理器做了：
django.contrib.auth.context_processors.auth：增加了user 和perms两个变量
django.core.context_processors.debug，若setting里DEBUG=True，增加了debug=true，sql_queries是{'sql': ..., 'time': ...} 的list
django.core.context_processors.media/static 增加了 MEDIA_URL/STATIC_URL，值和setting里的一样。
django.core.context_processors.csrf：增加了csrf_token
django.contrib.messages.context_processors.messages：messages ：消息的列表 DEFAULT_MESSAGE_LEVELS：消息层级的数字表示（1.7新增）

---
######自定义上下文处理器：
根据定义接收request对象，返回字典。然后加到 TEMPLATE_CONTEXT_PROCESSORS 或是上文提到的作为RequestContext的第三个参数(list中的元素)

#####加载模板
TEMPLATE_DIRS  模板文件绝对路径的tuple，django从这些路径里加载文件。即使在windows上，也要用'/'分隔路径。
```python
#注意末尾没有 /
TEMPLATE_DIRS = (
    "/home/html/templates/lawrence.com",
    "/home/html/templates/default",
)
```
#####python api
django.template.loader有两个函数，`get_template(template_name[, dirs])`和`select_template(template_name_list[, dirs])`，dir参数是1.7新加的，为了覆盖TEMPLATE_DIRS的设置。
两者都返回编译后的模板，即Template对象。`get_template`接收一个模板名，`select_template`接收模板名list。这两个函数，根据TEMPLATE_DIRS（或dir参数）和提供的模板名，拼接文件名，访问，编译，找到第一个模板为止，否则产生模板不存在的异常。

---
用子文件夹
这种方式是支持的，并且是建议的。将模板按app进行分类，这样比直接放在一个template文件夹下要好。`get_template('news/story_detail.html')`。注意news前面没有斜杠。

#####加载器的种类
默认用基于文件系统的loader。在setting通过 TEMPLATE_LOADERS 设置。
class filesystem.Loader:在 TEMPLATE_LOADERS 这样写：`django.template.loaders.filesystem.Loader`
根据 TEMPLATE_DIRS 加载。
class app_directories.Loader：查找每个 INSTALLED_APPS下的templates文件夹，从这里加载。如`INSTALLED_APPS = ('myproject.polls', 'myproject.music')`
`get_template('foo.html')`，会从`/path/to/myproject/polls/templates/`,`/path/to/myproject/music/templates/`找foo.html的文件，找到第一个则返回。
所以INSTALLED_APPS里个app名称的顺序就比较重要了。
这个loader默认开启。且在第一次被import时缓存了有templates子文件夹的 app。
class eggs.Loader：从egg文件夹下找（应该是templates子文件夹？文档没有写，但是说just like app_directories）。默认关闭。
django.template.loaders.cached.Loader模块：
class cached.Loader：
默认情况下，模板系统会在需要渲染模板时每次读取和编译，尽管很快，还是可以减少这部分开支。
这个加载器包装其他加载器，当第一次遇见时，用其他加载器，编译过之后存在内存里，后续请求就从内存里获得编译的Template对象。
```python
TEMPLATE_LOADERS = (
    ('django.template.loaders.cached.Loader', (
        'django.template.loaders.filesystem.Loader',
        'django.template.loaders.app_directories.Loader',
    )),
)
```
注意，所有自带的模板标签对使用cached loader是安全的，但是如果用自定义的或第三方的，要确保每个标签的Node实现是线程安全的，参考[template tag thread safety considerations](https://docs.djangoproject.com/en/1.7/howto/custom-template-tags/#template-tag-thread-safety)
默认关闭此加载器。
django根据TEMPLATE_LOADERS 按顺序查找对应的加载器，直到找到为止。

---
模板来源
TEMPLATE_DEBUG =True，每个Template对象又一个origin属性，记录是从哪个模板来的。
class loader.LoaderOrigin：
从模板加载器创建的Templates对象会用 django.template.loader.LoaderOrigin 类。name 模板文件路径 loadname传递给加载器的相对路径。
class StringOrigin：
从Template类创建的会用django.template.StringOrigin 类，source属性是创建模板的字符串。

---
#### render_to_string 捷径方法
loader.render_to_string(template_name, dictionary=None, context_instance=None)
为了减少重复性，提供了捷径方法：
```python
from django.template.loader import render_to_string
rendered = render_to_string('my_template.html', {'foo': 'bar'})
```
render_to_response() 正是调用了这个方法，来生成HttpResponse对象。

---
####将模板系统配置成单独模式
这部分是为了让你用模板系统作为其他应用的输出组件。
需要参考[Using settings without setting DJANGO_SETTINGS_MODULE](https://docs.djangoproject.com/en/1.7/topics/settings/#settings-without-django-settings-module),仅导入模板系统合适的模块，调用 `django.conf.settings.configure()`，指定你想指定的设置，包括但不限于 TEMPLATE_DIRS ，DEFAULT_CHARSET（默认utf8），以及任何以TEMPLATE开头的。

---
####用其他的模板语言
像jinja2等，` Context`对象和`render_to_response()`还可以继续使用。
django的Template接口很简单，构造函数接收字符串，一个render方法，接收Context对象。
如果其他模板语言的render方法接收字典而非Context对象，可以做如下包装：
```python
import some_template_language
class Template(some_template_language.Template):
    def render(self, context):
        # flatten the Django Context into a single dictionary.
        context_dict = {}
        for d in context.dicts:
            context_dict.update(d)
        return super(Template, self).render(context_dict)
```
下一步是编写一个Loader类，返回我们自定义的模板类而不是默认的Template。自定义Loader需要继承自`django.template.loader.BaseLoader`并且复写`load_template_source() `方法，这个方法接收一个`template_name `参数，从磁盘上加载模板，返回tuple：(template_string, template_origin).
Loader类的`load_template`方法通过调用`load_template_source`获取模板字符串，从template source实例化一个Template对象，返回一个tuple：(template, template_origin)。因为这是实际生成Template的方法，要用自定义模板类复写它。可以继承`django.template.loaders.app_directories.Loader`来利用它的`load_template_source`实现。
```pyton
from django.template.loaders import app_directories
class Loader(app_directories.Loader):
    is_usable = True

    def load_template(self, template_name, template_dirs=None):
        source, origin = self.load_template_source(template_name, template_dirs)
        template = Template(source)
        return template, origin
```
最后，要修改工程配置，用自定义的Loader。这样就可以用自定义的模板了。
