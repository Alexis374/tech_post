js design patten:
+ 由于js的灵活性，完成一种任务通常不止一种方法，最简单的是像C一样，过程式编程，将程序划分为函数。
+ 稍微复杂一点，定义一个类，并在原型上定义方法
+ 内省（运行时检查对象的所有属性方法）和反射（根据内省的信息动态实例化对象，调用其方法）。
+ 接口： 规定了对象应该具有的方法，但不负责具体实现。有了接口之后，可以按对象提供的特性进行分组；例如一个函数可以以一个接口为参数，任何实现了该接口的对象都可以作为参数。彼此不想关的对象也可以同等对待。
+ js实现接口方法： 1.注释描述 2.duck type模仿接口（重点）
+ duck type思想：对象具有接口定义的所有方法即可认为实现了该接口。Interface类定义接口名和必须的方法，ensureimplements函数是Interface类的类方法（直接定义在Interface而非其prototype上）接受类名以及实现的接口名，检查类名是否真正具有接口定义的方法。
+ 使用：检查代码中以**对象为参数**的方法，搞清正常运转时需要哪些方法；为每个不同的方法集建立Interface对象。在函数中提出针对构造器的检查，以ensureInmplements取代。
```js
var Interface = function(name,methods){
	this.name = name;
	this.methods = [];
	for (var i=0,len = methods.length;i<len;i++){
		if(typeof methods[i] === 'string'){
			this.methods.add(methods[i])
		}
		else{
			alert(methods[i]+" is not a method string");
		}
	}
}
Interface.ensureInplements = function(obj,interfacename){
	if(arguments.length)<2{
		throw new Error('at least 2 arguments')
	}
	for(var i=1,len = arguments.length;i<len;i++){
		var interface = arguments[i];
		if(interface.constructor !==Interface){
			throw new Error('not a Interface');
		}
		for(var j=0;methodsLen = interface.methods.length;j<methodsLen;j++){
			var method = interface.methods[j];
			if(!obj[method] ||typeof obj[method]!=='function'){
				throw new Error('method'+method+'not found');
			}
		}
	}
}

var DynamicMap = new Interface('DynamicMap',['zoom','draw']);

function displayRoute(mapInstance){
	Interface.ensureInplements(mapInstance,DynamicMap);
	mapInstance.zoom(5);
	mapInstance.draw();
}
```
+ 依赖接口的设计模式：工厂，组合，装饰者，命令。

=======
+ 封装和信息隐藏:
闭包：
```
function foo(){
	var a =10;
	function bar(){
		a *=2;
		return a;
	}
	return bar
}
var baz = foo();
baz()  // =>  20
baz() // => 40
baz(); // => 80

var blat = foo(); //a reference to another bar
blat() //=> 20
```
+ js作用域是函数级的。
+ js作用域是词法性的（lexical),**函数是运行在定义它们的作用域中（foo）而不是运行在调用它们的作用域中**，
bar在foo中定义，就能访问foo中的定义的所有变量，即使foo已经执行结束。
+ foo返回后其作用域被保存，但只有它返回的函数能访问foo的作用域。foo两次调用生成了两个作用域，
所以baz blat各自拥有作用域的一个副本，故访问的a是不同的
+ 用闭包实现私有函数：
```
var Book  = function(){
	var title,author,isbn;   // private variable
	
	function checkIsbn(isbn){ //private function
		if(isbn.length>10)
			return true;
		return false;
	}

	this.setIsbn(isbn) = function(){  //privileged methods.特权方法，可以在外部被访问
		if(checkIsbn(isbn)){
			this.isbn = isbn;
		}
		else{
			throw new Error('Invalid ISBN')
		}
	}
	this.getIsbn(isbn) = function(){
		return isbn;
	}
}
Book.prototype = {
	display:function(){
		console.log(this.getIsbn()) //任何不需要直接访问私有属性的函数可以放在类的原型中
	}
}
```
弊端：每个对象都会有自己的私有方法和特权方法，占内存。 不利于派生子类，子类不能访问超类的私有属性方法
+ 静态方法和属性
```
var Book = (function(){
	// private static attributes               由于定义后立即调用，私有的静态属性只会存在一份
	var numOfBooks = 0;					私有方法是否要设置成静态的，关键要看是否访问实例数据
	// private static method
	function checkIsbn(isbn){
		...
	}




	//return the constructor
	var Book =  function(newIsbn,newTitle,newAuthor){
		//private attributes
		var isbn,title,author;

		//privileged method
		this.getIsbn = function(){
			return isbn
		}
		this.setIsbn = function(newIsbn){
			if(checkIsbn(newIsbn))
				isbn = newIsbn;
			else
				throw new Error('Invalid isbn')
		}
		...
		// constructor code
		numOfBooks++;
		this.setIsbn(newIsbn);
		this.setTitle(newTitle);

	}
	var Constant = {
		UPPER_BOUND:100;
		LOWER_BOUND:-100;
	}

	//privileged static method
	Book.getConstant = function(name){   // 特权静态方法，用于访问常量
		return contants[name];
	}
	return Book
})();
// public static method
Book.convertToTitleCase = function(){ //共有静态方法用于与这个类整体相关的任务，而非这个类实例的任务，不依赖实例对象的数据
	...
}
// public method	
Book.prototype = {
	display:function(){
		console.log(this.getIsbn()) //任何不需要直接访问私有属性的函数可以放在类的原型中
	}
}

Book.getConstant('UPPER_BOUND');
```
+ 封装：利： 屏蔽细节，弱化耦合；弊：复杂性，难单元测试（=>只测公有方法）

======
+ 继承：
