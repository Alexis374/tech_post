####jquery note
+ 选择器
	+ 常规的css选择器，如 id，class，元素，*，','（并列选择），'ance desc'(选取祖先是ance的desc)，'parent>child'(选取父元素是parent的child)，'prev+next'(选next，这个next的前面兄弟**元素**元素是prev),'prev~siblings'(选取与prev相邻的，在prev之后的所有sibling元素)
	+ 过滤性选择器,类似伪类：':first'(只返回一个),':eq(index)',':contains(text)'(文本中含有text的元素),':has(selector)'(内部拥有另一个选择器selecotr)，':hidden'(获取不可见的元素，如p:hidden),':visible'(获取可见元素)，'[attribute=value]'(有属性是value的元素)，同理有'[attr!=val]'(不等于),'[attr*=val]'(attr包含val)，':first-child'（选取第一个子元素的集合）
+ 表单选择
	+ ':input表单选择器获取表单元素，可以获得select，button等','input'只能获得`input`元素
	+ ':text'获取单行的文本输入框元素，不包括`textarea`
	+ ':password'
	+ ':radio' 获取单选按钮  等价于 'input[type=radio]'
	+ ':checkbox' 等价形式同上
	+ ':submit' 等价形式同上
	+ ':image' , ':button' , 
	+ ':checked' ':selected' 两者同上。只是对应的是html标签内本来就有 checked="checked"
+ attr() |set get	
+ html() text(),html()方法可以获取元素的HTML内容，因此，原文中的格式代码也被一起获取，而text()方法只是获取元素中的文本内容，并不包含HTML格式代码
+ addClass(),css()|get set , removeAttr(),removeClass()
+ selector.append(content) 等价于 $(content).appendTo(selector)
+ selector.after(content)  意思是 selector之后是content；selector.before(content),意思是 selector的前面是content
+ selector.clone()
+ $(selector).replaceWith(content)和$(content).replaceAll(selector)
+ $(selector).wrap(wrapper) 如$(".red").wrap('<div></div>') div 包裹在.red外 和$(selector).wrapInner(wrapper)  如$(".red").wrapInner('<i></i>'); i被包括在.red内部
+ $(selector).each(function(index))
+ remove(selector)方法删除所选元素本身和子元素，该方法可以通过添加过滤参数指定需要删除的某些元素，而empty()方法则只删除所选元素的子元素。

+ 事件：
	+ ready事件 `$(document).ready(function())` == `$(function(){})`
	+ hover,`$(selector).hover(over，out);`移上时执行over函数，移出时执行out函数
	+ bind `$(selector).bind(event,[data] function)` 可以绑定多个事件，以字符串的形式空格隔开
	+ `$(selector).toggle(fun1(),fun2(),funN(),...)`每点击一次，在函数之中循环执行
	+ `$(selector).unbind(event,fun)`,参数event表示需要移除的事件名称，多个事件名用空格隔开，fun参数为事件执行时调用的函数名称.unbind() 方法会删除指定元素的所有事件处理程序。
	+ one 绑定一次性事件 `$(selector).one(event,fun)`
	+ 