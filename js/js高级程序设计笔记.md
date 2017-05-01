+ try-catch:
	+ catch后的括号不能为空，即必须指定一个异常对象，有`name`和`message`属性。
	+ `try-catch-finally` 在函数中，try块的代码执行正确且包含return语句：
		+ 若`finally`没有return 而有其他代码，执行其他代码，并返回try中return的值；
		+ 若`finally`含有return语句，则返回`finally`的值
		+ 经测试python js在上述情况下表现一致
	+ `throw new Error(message-string)`
	+ 只捕获代码编写者确切知道如何处理的异常，避免浏览器以默认方式处理，抛出错误的目的在于提供详细原因
+ 未经处理的异常会产生`error`事件，其事件只能是DOM0级的



-------------------
### 对象
对象属性的特性(the attribute of property)
1. 数据属性：
	+ `[[Configurable]]` 能否删除属性以及修改其他的特性，或能否改为访问器属性。default：true
	+ `[[Enumerable]]` for-in 能否列举 default：true
	+ `[[Writable]]` 能否修改属性的值 default：true
	+ `[[Value]]`  属性的值，default：undefined
针对特定的属性设定其特性：
```js
var person = {};
Object.defineProperty(person,"name",{
	writeable:false,
	value:"TEST"
});
```
**在Object.defineProperty中不指定特性时，默认都是false，value是undefined**

2.访问器属性
	+ `[[Configurable]]` 能否删除属性以及修改其他的特性，或能否改为数据属性。default：true
	+ `[[Enumerable]]` for-in 能否列举 default：true
	+ `[[Get]]` 读取时调用的函数 default：undefined
	+ `[[Set]]` 写入时调用的函数 default：undefined
常用在设置一个属性的值导致其他属性变化（Vue.js）。

`Object.defineProperties()` 简化属性的赋值，一次可以定义多个属性。
`Object.getOwnPropertyDescriptor(obj,propName)` 可以得到定义的描述符
-------------------
对象创建：
1. 很”low“的方式是创建一个工厂函数，内部新建一个对象，设置属性之后return这个对象
2. 构造函数。new 构造函数的执行过程：创建一个对象，并将this指向这个对象，设置属性，构造函数执行完之后隐式返回这个对象（不必return）。**此方式的得到的对象 constructor指向构造函数**
```js
function Person(){}
var p = new Person()
p.constructor === Person
p instanceof Person
p instanceof Object
```
构造函数不用new调用时，当做普通函数，可以给它动态绑定作用域（call,apply）或直接调用（this指向全局变量）
存在的问题：同一类对象的方法是不同的函数对象，浪费内存
3.原型模式
+ js中几乎所有对象都有原型（Object.prototye没有。）
+ 上例中，构造函数（Person）才有prototype属性，指向所生成对象（p）的原型。
+ p的原型有constructor属性，指向构造函数（person），以及从Object等继承来的属性和定义在Person.prototype上的属性
+ 每个对象都有个`[[Prototype]]`指针，指向原型，但不可直接访问.某些浏览器可以通过`p.__proto__`访问原型。
+ 当用p.prop属性访问对象时，先在对象中找，找不到则在原型上找prop属性，若找不到则沿着原型链向上查找。
+ usefule functions:
	+ `Object.getPrototypeOf(p)`  //return p的原型
	+ `p.hasOwnProperty(propName)` //自己是否有prop属性
	+ ``
+ 重写原型属性的注意事项(`Person.prototype = {}`的形式)：
	+ 需要显式设置`Person.prototype.constructor=Person`;调用`Object.defineProperty(Person.prototype,'constructor',{value:Person,enumrable:false})`更好
	+ 会切断已生成的对象和新原型的联系。
	
4. 寄生构造函数
在构造函数内新建一个对象，设置属性，然后返回这个对象
```js
function SpecialArray(){ 
	//创建数组
	var values = new Array(); 
	//添加值
	values.push.apply(values, arguments); 
	//添加方法
	values.toPipedString = function(){ 
	return this.join("|"); 
	}; 
	//返回数组
	return values; 
} 
var colors = new SpecialArray("red", "blue", "green"); ///不用new也可以
alert(colors.toPipedString()); //"red|blue|green"
colors.constructor === Array
(colors instanceof Array)===true
(colors instanceof SpecialArray)===false
```
5. 稳妥模式
类似上面一种，但不适用this，不使用new 创建对象
```js
	function Person(name, age, job){ 
		//创建要返回的对象
		var o = new Object();
		//可以在这里定义私有变量和函数
		//添加方法
		o.sayName = function(){ 
		alert(name); 
		}; 
		//返回对象
		return o; 
}
var friend = Person("Nicholas", 29, "Software Engineer"); 
friend.sayName(); //"Nicholas" 
``` 
