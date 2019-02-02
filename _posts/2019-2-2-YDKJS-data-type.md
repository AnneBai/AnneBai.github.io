---
layout: post
title: 再看数据类型
---
## 数据类型
在JavaScript中, 主要基本数据类型有七种:

+ string
+ +number
+ boolean
+ null
+ undefined
+ object
+ symbol(ES6新增);

object是对象, 是引用类型的数据结构, 可以动态添加属性; 其他六种都为原始基本类型, 不能添加属性;

对象是一组无序键值对的集合; 一般可以通过对象字面量或者使用构造函数的方法创建. 

JavaScript中有一些内建对象, 均为对象的子类型, 它们是可以构建相应子类型对象的构造函数:

+ String
+ Number
+ Boolean
+ Object
+ Function
+ Array
+ Date
+ RegExp
+ Error; 

```
"str" instanceof String  // false
new String("str1") instanceof String // true
typeof "str" // "string"
typeof new String("str1") // "object"
```
string, boolean, number原始基本类型是
**不可变的字面值**, 自身没有属性和方法, 但是依然可以对它们使用一些特有的操作如访问字符串长度, 查询某个字符的位置等, 因为JavaScript引擎可以在需要的时候把它们++转换为对应的对象类型++.

例如字符串会被转换为"String"对象类型, 当访问或操作执行完毕, 再还原为原始基本类型. 相当于在需要的时候临时包装成对象, 但++只发生在操作执行的过程++中.

`null`和`undefined`仅有基本类型值即它们自己, 没有对应的构造对象形式;

Object, Function, Array, RegExp类型不论使用字面量还是构造函数创建都是对象;
Error对象一般在抛出异常时自动创建, 很少需要在代码中显式创建;
Date只能通过构造函数来创建, 因为没有对应的字面形式;

Symbol类型的数据通过Symbol函数创建, 但和其他构造函数不同, Symbol函数不能使用`new`操作符, 也没有包装的对象类型.
```
var prop = {a: Symbol(), b: Symbol()};
var obj = {};
obj[prop.a] = 1;
obj[prop.b] = 2;
obj // {Symbol(): 1, Symbol(): 2}
Object.getOwnPropertySymbols(obj) // [Symbol(), Symbol()]
```
一般的属性遍历方式: `for...in/of`, `Object.keys()`, `Object.getOwnPropertyNames()`不会包含Symbol; `JSON.stringify()`返回值也不会包含Symbol. 可以通过`Object.getOwnPropertySymbols`获取所有Symbol属性名数组;

既可以用字面量也可以用构造函数形式创建的情况下, 目前推荐的最佳实践是尽可能地使用字面量语法创建对象.因为更简单和直观.