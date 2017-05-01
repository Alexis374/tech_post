#### Haskell入门
###### 本篇是浏览[http://learnyoua.haskell.sg/](learnyoua.haskell.sg)所做笔记，作为haskell的入门笔记
----
+ haskell 静态类型(statically typed)，自动类型推导(type inference),惰性求值，引用透明
	+ 引用透明：以同样的参数调用同一个函数多次，所得结果完全一样
	+ 惰性求值：DoubleMe(DoubleMe(DoubleMe(xs)))只会对xs遍历一次
+ type `ghci` `:set prompt "ghci> "`
+ 逻辑操作符有`&&`,`||`,`not`;值有`True`,`False`;判断相等，不等`==`,`/=`
+ `*`,`+`这样的在命令式语言中的操作符也是函数，不过是中缀函数，即参数在函数的两边
+ haskell中的前缀函数调用方式： `函数名 空格分隔的参数列`
```haskell
ghci> succ 8
9
ghci> min 9 10
9
ghci> min 3.4 3.2
3.2
```
+ 函数调用的优先级最高
```haskell
ghci> succ 9 + max 5 4 + 1
16
ghci> (succ 9) + (max 5 4) + 1
16
```
+ 定义函数（函数名 参数列=行为）
```haskell
doubleMe x = x + x
```
+ if-then-else:必须是这样的结构，if语句要返回值的，返回的是then的值或else的值。
```haskell
doubleSmallNumber' x = (if x > 100 then x else x*2) + 1
#不加括号，则+1只在else语句里生效
```
+ 无参数的函数称之为定义（definition），`conanO'Brien = "It's a-me, Conan O'Brien!"` 两者即等价

-------
+ list中所有元素必须是同一类型的
+ list:`[1,2,3,4]` 字符串是list，`"hello"`是['h','e','l','l','o']的语法糖
+ 合并`lista ++ listb`，追加一个元素用冒号运算子：`ele:list`
+ `[1,2,3]` 实际上是 `1:2:3:[]` 的语法糖。
	+ 个人理解：体现了惰性运算的性质。`:`的操作数类型为ele和list，第一个冒号1:2出现时后一个参数不是list，要延迟计算，
	一直延迟到最后3:[]，结果是[3],然后向前计算

+ 索引运算子`!!` 
```haskell
ghci> "Steve Buscemi" !! 6  
'B'  
ghci> [9.4,33.2,96.2,11.2,23.25] !! 1  
33.2
```

+ List 中的 List 可以是不同长度，但必须得是相同的类型。如字符串的长度可以不同 `let pl = ["js","python","c++"]`
+ 常用操作： 比较大小`> < >= <= ==`(从第一个元素开始，相等则比较下一个元素)；`head tail last init null reverse`,`take num list`返回list的前num个元素；`drop num list`删除前num个元素，`maximum sum`;`elem` 判断是否在list内，`10 `elem` [3,4,5,6]  `(注意包围在"`"之中)
+ range：，上限包括在内，与python略有不同，python不包括上限
```haskell
ghci> [1..20]
[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20]
ghci> ['a'..'z']
"abcdefghijklmnopqrstuvwxyz"
ghci> ['K'..'Z']  
"KLMNOPQRSTUVWXYZ"
```
逗号将前两个元素隔开，再标上上限
```haskell
ghci> [2,4..20]
[2,4,6,8,10,12,14,16,18,20]
ghci> [3,6..20]
[3,6,9,12,15,18]
ghci> [100,96..(-1)] //-1要加括号，否则运算优先级出错
[100,97,94,91,88,85,82,79,76,73,70,67,64,61,58,55,52,49,46,43,40,37,34,31,28,25,22,19,16,13,10,7,4,1]
```
+  `cycle` 接受一个 List 做参数并返回一个无限 List,`repeat` 接受一个值作参数，并返回一个仅包含该值的无限List。惰性求值，只求需要求到的部分; `replicate num element`,返回num个相同元素的list如`replicate 3 10`
+ list comprehension
[对元素进行的操作| 过滤选出元素 ]，如`[ x*y | x <- [2,5,10], y <- [8,10,11]]`，`<-`表示迭代list中的元素
+ tuple
	+ tuple的类型取决于item的数目和各自的类型，不同item类型可以不同
	+ tuple不能为一个元素
	+ 2个元素的tuple成为pair，有专有的`fst` `snd`函数
