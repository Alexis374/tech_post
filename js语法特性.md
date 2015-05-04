####js语法特性

+ `null` 和 `undefined`
	+ 只声明未初始化的变量返回`undefined`
	+ 数值类型环境中undefined值会被转换为NaN  `var a;a + 2 = NaN`
	+ 空值null在数值类型环境中会被当作０来对待，而布尔类型环境中会被当作false.`var n = null;console.log(n * 32); // logs 0`
	+ 无返回值的函数返回undefined
+ 全局变量实际上是全局对象的属性。浏览器中，全局对象缺省是window
+ 声明提升：你可以引用稍后声明的变量，而不会引发异常。注意仅仅是**不引发异常**而已， 变量感觉上是被“举起”或提升到了所有函数和语句之前。但是，未被初始化的变量仍将返回 undefined 值
```
//example1
console.log(x === undefined); // logs "true"
var x = 3;  // 声明提升，不返回异常，但上句仍是 undefined

//example2
var myvar = "my value";
(function() {
  console.log(myvar); // undefined
  var myvar = "local value";   //相当于此句声明了myvar 局部变量。
})();

//example3  与2区别看
var myvar = "my value";
(function() {
  console.log(myvar); // "my value" 这句进行了属性查找，使用的是全局的myvar值
})();

```
+ `parseInt`,'parseFloat'和'+'
前两者会从参数字符串开始进行解析，直到遇到无法解析为数字的部分，然后返回。若解析失败则返回NaN；
后者只要含有不能解析为数字的就返回NaN。
注意包含 `1e5`这种格式的可以解析为10000
+ `NaN == NaN`结果为false，用isNaN判断。isFinite() 来判断一个变量是否为 Infinity, -Infinity 或 NaN
+ 数组Array方法：
	+ a.join(sep)，与python 的 sep.join(a)刚好相反
	+ a.slice(start,end) 类似python的切片
	+ a.pop()删除并返回最后一个元素，a.shift()删除并返回第一个元素.push unshift则在对应位置插入，并返回插入后元素个数
	+ a.splice(start, delcount[, item1[, ...[, itemN]]]) 从 start 开始，删除 delcount 个元素，然后插入所有的 item。
+ 函数调用时参数可以和声明时不同。多的被忽略，少的当成undefined；函数参数是访问了函数中的`arguments`对象。这个对象类似数组。
+ 函数是对象。可以在函数上运用apply（接受参数数组）或call（接受单个的参数）方法，进行作用域绑定。
```
function avg() {
    var sum = 0;
    for (var i = 0, j = arguments.length; i < j; i++) {
        sum += arguments[i];
    }
    return sum / arguments.length;
}

avg.apply(null,[2,3,4,5,6]);
avg.call(null,2,3,4,5,6,7);

```
+ `apply`,`call`函数的第一个参数是要绑定的对象，如func.call(obj,arg1,arg2),函数func内部的this指代的就是obj。上例中avg函数内没有this，可以不传递。
+ 当func中有this，而传给apply或call的对象是null/undefined时，则内部的this指的是全局对象
+ new 关键字，创建一个空对象，并使函数内部的this指向这个对象，在这个对象上进行增加属性。
+ 共有的方法加入到原型中。
+ 闭包。闭包指有权访问另一个函数作用域中变量的函数。