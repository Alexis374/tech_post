###[CSRF保护](https://docs.djangoproject.com/en/1.7/ref/contrib/csrf/#django.views.decorators.csrf.ensure_csrf_cookie)
---
CSRF中间件和模板标签提供了易用的对于[跨站请求伪造](http://www.squarefree.com/securitytips/web-developers.html#CSRF)的保护。这种攻击在恶意网站包含链接，按钮或利用已登录你的网站的，浏览恶意网站的用户的证书，意在在你的网站上实施某些动作的js代码。也包括，相关类型的攻击，‘登录CSRF’，这种攻击是由攻击站点追踪用户用某些凭证登录网站。
对CSRF的第一层保护是确保GET请求（和其他‘安全’方法）是无副作用的。通过非安全方法的请求，如POST，PUT，DELETE，可以由下列步骤保护。

####怎么使用
1. 增加中间件 `django.middleware.csrf.CsrfViewMiddleware` （必须放在 其他假定CSRF已经处理的中间件之前）或者你可以使用 `csrf_protect() `装饰器，修饰想保护的view
2. 在所有POST到内部URL的表单中，在`<form>`内加入 `{% csrf_token %}`。外部URL不用加，否则会导致泄露
3. 在对应的函数中，确保`django.core.context_processors.csrf`上下文处理器被使用。通常，可以由以下方法做到：
    1. 用`RequestContext`，这会一直使用 `django.core.context_processors.csrf`，不管你的上下文处理器是怎么设置的，如果你用的是通用视图或contrib应用，这已经包含了，因为这些应用已经用了`RequestContext`。
    2. 人工import和使用这个处理器来产生`csrf_token`,并加入模板上下文，如：
    ```python
    from django.core.context_processors import csrf
    from django.shortcuts import render_to_response
    
    def my_view(request):
    c = {}
    c.update(csrf(request))
    # ... view code here
    return render_to_response("a_template.html", c)
    ```
    你可能自己写这个包装函数，来包含这些步骤。

####AJAX
尽管上述方法可以用于AJAX POST请求，但不方便：需要记得在每个POST请求中传递CSRF token。另一种方法：在每个XMLHttpRequest上，设置自定义的`X-CSRFToken`，值是csrf token的值。这通常简单，因为许多js框架提供了允许设置每个request头的钩子。
第一步，需要得到CSRF token值，推荐的方法是csrftoken cookie，这个cookie在设置了csrf保护的时候会开启。默认是cookie名为`csrftoken`，可以设置`CSRF_COOKIE_NAME`为指定字符串来改变。
用jQeury得到token的方法：
```js
function getCookie(name) {
    var cookieValue = null;
    if (document.cookie && document.cookie != '') {
        var cookies = document.cookie.split(';');
        for (var i = 0; i < cookies.length; i++) {
            var cookie = jQuery.trim(cookies[i]);
            // Does this cookie string begin with the name we want?
            if (cookie.substring(0, name.length + 1) == (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}
var csrftoken = getCookie('csrftoken');
```
可以用Jq的cookie插件简化：`var csrftoken = $.cookie('csrftoken');`
注意： 如果显式地在tempalte里包含`{%csrf_token%}`,token同样也在DOM中.cookie包含着权威的token，`CsrfViewMiddleware`更倾向于使用cookie中的token，不管怎样，你应该使用cookie中的token
警告：如果你的view函数渲染的是没有使用`{%csrf_token%}`的模板，django不会设置token cookie，这类情况经常发生在表单动态加入到页面中。为了强调这种情况，Django提供了`ensure_csrf_cookie()`装饰器强制设定cookie。

最后，你需要在Ajax请求头中设置header，但要注意不要将token发送到其他域中。
```js
function csrfSafeMethod(method) {
    // these HTTP methods do not require CSRF protection
    return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
}
function sameOrigin(url) {
    // test that a given url is a same-origin URL
    // url could be relative or scheme relative or absolute
    var host = document.location.host; // host + port
    var protocol = document.location.protocol;
    var sr_origin = '//' + host;
    var origin = protocol + sr_origin;
    // Allow absolute or scheme relative URLs to same origin
    return (url == origin || url.slice(0, origin.length + 1) == origin + '/') ||
        (url == sr_origin || url.slice(0, sr_origin.length + 1) == sr_origin + '/') ||
        // or any other URL that isn't scheme relative or absolute i.e relative.
        !(/^(\/\/|http:|https:).*/.test(url));
}
$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (!csrfSafeMethod(settings.type) && sameOrigin(settings.url)) {
            // Send the token to same-origin, relative URLs only.
            // Send the token only if the method warrants CSRF protection
            // Using the CSRFToken value acquired earlier
            xhr.setRequestHeader("X-CSRFToken", csrftoken);
        }
    }
});
```

####其他模板引擎
使用其他模板引擎时，手工添加csrf_token,如CHeetah中
```html
<div style="display:none">
    <input type="hidden" name="csrfmiddlewaretoken" value="$csrf_token"/>
</div>
```
---
####装饰器方法
除了加中间件这种没有限制的做法，还可以使用`csrf_protect`装饰器，在view（输出token）和template的post表单上（包含token）同时使用。
单独使用装饰器并不推荐，因为经常会遗忘。万无一失的策略是用两者，会招致最小的损失
```python
from django.views.decorators.csrf import csrf_protect
from django.shortcuts import render

@csrf_protect
def my_view(request):
    c = {}
    # ...
    return render(request, "a_template.html", c)
```
---
####拒绝请求
默认情况下，不满足csrf中间件检查会产生403错误，这会在真正的攻击，或者忘了在POST表单添加token 发生时出现。
这并不友好，可以自己设定处理这种错误的view函数，要设置` CSRF_FAILURE_VIEW`

---
####如何工作
CSRF保护基于下列：
1. CSRF cookie设置为随机值，是与session无关的，其他站点不会访问到。是由中间件设定的，被设为永久有效，但是因为没有方法保证cookie不过期，每个调用了`django.middleware.csrf.get_token()`的方法都会发送csrftoken的cookie
2. 一个隐藏的表单‘csrfmiddlewaretoken’被加到post表单里，值时csrf cookie的值。这是由template tag`{%csrf_token%}`完成的
3. 对于所有非安全的HTTP方法，csrf cookie必须被发送，`csrfmiddlewaretoken`字段必须正确，否则403。这由中间件检查
4. 对于HTTPS请求，中间件执行严格的referer检查，强调在https下可能发生中间人攻击很有必要。中间人攻击在用session无关的场景下，因为http的Set-Cookie头被与网站通过https交流的客户端接收。（Referer check不会在http的情况下做，因为http下 referer header的存在不充分可信）。

这确保了，只有来自你网站上的表单可以被用作POST数据。

故意忽略了GET请求（和其他安全方法），这些方法应该不含有任何危险的副作用，所以利用GET的CSRF攻击无害。RFC定义了POST PUT DELETE为非安全的。为了最大化的保护，其他的方法被认为是非安全的。

---
缓存
如果csrf_token 模板标签被模板使用了（或者get_token函数被调用了），中间件会增加cookie和Vary:Cookie头，这意味着，如果按指导使用（`UpdateCacheMiddleware`在最前面），cache中间件和csrf中间件可以良好工作。
但是，如果你在个别views上用了cache 装饰器，csrf中间件不能产生cookie和Vary头，response就会被缓存但不携带任何一个。这种情况下，在需要csrftoken的view上，你需要首先使用`django.views.decorators.csrf.csrf_protect() `
```python
from django.views.decorators.cache import cache_page
from django.views.decorators.csrf import csrf_protect

@cache_page(60 * 15)
@csrf_protect
def my_view(request):
    # ...
```
---
####测试
因为需要每个post请求都要有csrftoken，csrf中间件是测试的最大障碍。Django的http 测试client设置了flag，使测试使用中间件和`csrf_protect`装饰器的view轻松了，不再拒绝请求。在其他方面(如发送cookie)表现一样。
如果你需要测试CSRF检查，可以创建一个强制检测的实例：
```py
>>> from django.test import Client
>>> csrf_client = Client(enforce_csrf_checks=True)
```
---
####限制
子域名可以设置整个域的cookie。通过设置cookie，用对应的token，子域名可以绕过CSRF保护。唯一避免的方法是子域名被信任的用户控制（或，至少不能设置cookie）注意即使没有CSRF，也会有其他漏洞，如session fixation，这使得将子域给不信任的用户是不好的，而且漏洞不会被当前浏览器轻易修复。

---
####边界情况
某些view有不寻常的需求，不适合这里设想的情况。许多工具可以在此时利用。
####工具
`csrf_exempt(view)`免除csrf保护
```python
from django.views.decorators.csrf import csrf_exempt
from django.http import HttpResponse

@csrf_exempt
def my_view(request):
    return HttpResponse('Hello world')
```
`requires_csrf_token(view)` 通常 若csrf中间件的`process_view`或等价的`csrf_protect`没有执行时，csrf_token模板标签不会工作，这个函数会确保模板标签工作，这个装饰器和`csrf_protect`工作相同，但不拒绝到来的requst
```python
rom django.views.decorators.csrf import requires_csrf_token
from django.shortcuts import render

@requires_csrf_token
def my_view(request):
    c = {}
    # ...
    return render(request, "a_template.html", c)

```

`ensure_csrf_cookie(view)`强制view发送CSRF cookie

---
####场景
#####CSRF protection should be disabled for just a few views
用` csrf_exempt()`
##### CsrfViewMiddleware.process_view not used
可能遇到404或500，但你在表单离仍然需要CSRF token。use requires_csrf_token()
##### Unprotected view needs the CSRF token
某些view不需要保护，或被免除保护，但仍然需要csrf token。
用 csrf_exempt() followed by requires_csrf_token(). 即 requires_csrf_token是里面的装饰器
#####View needs protection for one path
 use csrf_exempt()整个view函数, and csrf_protect()指定需要保护的路径. 
```python
from django.views.decorators.csrf import csrf_exempt, csrf_protect
@csrf_exempt
def my_view(request):
    @csrf_protect
    def protected_path(request):
        do_something()

    if some_condition():
       return protected_path(request)
    else:
       do_something_else()
```
#####页面用了Ajax，没有html表单
Solution: use ensure_csrf_cookie() on the view that sends the page.

---
####可复用的app
因为开发者可能关闭csrf中间件，所有contrib app用了`csrf_protect` 装饰器，确保安全。建议制作其他可复用的app时也加上改装饰器

---
####设置

CSRF_COOKIE_AGE
CSRF_COOKIE_DOMAIN
CSRF_COOKIE_HTTPONLY
CSRF_COOKIE_NAME
CSRF_COOKIE_PATH
CSRF_COOKIE_SECURE
CSRF_FAILURE_VIEW