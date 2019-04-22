---
layout: post
title: 再看对象
---

## 对象
### 属性访问
对象可以包含属性, 属性名总是字符串, 存储在对象容器的内部, 但属性值不一定也存储于此, 属性名可以被看做指向属性值实际存储地的"指针"(或reference-引用).

对象属性的访问可以有两种形式(点访问和中括号访问):

```javascript
var obj = {
a: 1,
}
obj.a // 1
obj["a"] // 1
```

属性名符合标识符规则(简单说就是可以作为合法变量名), 可以使用点访问; 如果属性名是如"prop-a"这种包含不能用于标识符的字符, 则只能通过中括号并把属性名放在字符串引号里: `obj["prop-a"]`;

中括号访问还可以用于计算型属性, 在其中放表达式相当于先对表达式求值再作为属性名访问.
普通对象的属性名都是字符串, 如果中括号中的值不是字符串, 会先转为字符串; 如果是对象则可能转化后都会变为`obj["[object Object]"]`;

访问属性时, 会首先在对象自己拥有的属性中查找, 如果没有找到对应的值, 则会沿着原型链一级一级往上查找, 直到找到则返回该值, 最后还没有找到则返回`undefined`;

确定某个属性在对象的自身属性中存在(而不是在原型链上或者根本没有), 可以通过`obj.hasOwnProperty(propName)`的方式.

### 对象的复制
简单通过"="复制对象, 也只是简单的复制了一份引用, 与原来的变量名一样都指向那个对象;

如果使用`Object.assign()`方法或扩展运算符复制(或"扩展")对象, 对象中保留的属性如果是不可变的简单类型值会拷贝一个副本到当前对象, 如果是引用值则依然是和普通赋值相似, 拷贝一个引用,而不是重新复制整个引用类型值到当前对象. 

所以`Object.assign()`方法和扩展运算符都是对对象浅拷贝. 且它们只拷贝可枚举的(enumerable)直属属性(own property), 不会拷贝继承的属性.

```javascript
var obj = {a: 1, b: 2}
var obj1 = {o: obj};
var newObj = {...obj1};
newObj.o === obj // true
```

### 属性描述符
EcmaScript中定义一些只有内部使用的特性来描述属性的特征，在JS中不能直接访问它们。对"属性的属性"是通过属性描述符定义的:

```javascript
var obj = {
    a: 1,
};
Object.getOwnPropertyDescriptor(obj, "a");
// -> {value: 1, writable: true, enumerable: true, configurable: true} 
```

有两种属性, 数据属性和访问器属性; 

**数据属性**包含这个属性的数据值, 用来读取和写入, 其特性有:
+ `value`: 为当前属性对应的值;
+ `writable`: 是否可以修改(可写); 
+ `enumerable`: 可枚举性; 为true时则代表在`for...in/of`循环或`Object.keys()`等操作中可以被访问到, 否则会在属性枚举操作中隐藏.
+ `configurable`: 可配置性; 为true则可以使用`Object.defineProperty()`方法修改它的描述符定义. 也因此,设置为false是单向操作, 无法再设置回true, 唯一还可以设置的是writable由true变为false. 为false时也无法用`delete`操作符从对象中删除这个属性.

```javascript
Object.defineProperty(obj, "a", {
    value: 2,
    writable: false, 
    configurable: true,
    enumerable: true
});
obj.a = 3;
obj.a; // 2  修改不可写的属性会被忽略, 严格模式下则会报错(TypeError)

Object.keys(obj) // ["a"]
Object.defineProperty(obj, "a", {
	writable: true,
	configurable: true,
	value: 3,
	enumerable: false
});
Object.keys(obj) // []

Object.defineProperty(obj, "a", {
    // 可配置性设置为false, 但依然可写
	writable: true,
	configurable: false,
	value: 6,
	enumerable: true
});
Object.defineProperty(obj, "a", {
    // 修改value值可以生效, 修改enumerable或configurable都会报错(TypeError)
	writable: true,
	configurable: false,
	value: 7,
	enumerable: true
});
obj.a // 7

Object.defineProperty(obj, "a", {
    // writable变为false之后, value值也不能再重新设置, 否则会报错
	writable: false,
	configurable: false,
	value: 8,
	enumerable: true
})
// 报错 Uncaught TypeError: Cannot redefine property: a
```

**访问器属性**不包含数据值, 但包含一对儿getter和setter函数(非必须);访问器属性不能直接定义, 必须通过`Object.defineProperty()`来定义.

```javascript
var obj = {a: 1};
Object.defineProperty(obj, "b", {
    get: function () {
        return this.a + 1;
    },
    set: function (v) {
        this.a = v - 1;
    }
});
obj.b; // 2
obj.b = 4;
obj.a; // 3
```

使用访问器属性的常见形式是设置一个属性的值可以影响另一个属性发生变化。

使用`Object.defineProperties()`方法可以一次定义多个属性:

```javascript
var obj = {};
Object.defineProperties(obj, {
    a: { // 数据属性
        writable: true,
        value: 1,
    },
    b: { // 访问器属性
        get: function () {
            return this.a * 2;
        },
        set: function (v) {
            this.a = v / 2;
        }
    }
});
obj.b; // 2
obj.b = 6;
obj.a; // 3
```

### 防止扩展/封存/冻结
这些只是对对象自身的属性进行设置, 相当于是"浅封存"或"浅冻结".

1. 防止扩展(preventExtension): 仅保留已有属性, 不能再添加新属性

```javascript
Object.preventExtensions(obj);
obj.b = 5;
obj.b // undefined
```

2. 封存(seal): 相当于调用`Object.preventExtensions(..)`并把所有已有属性标记为`configurable: false`;

```javascript
var obj = {b: 3};
var o = {a: obj};
Object.seal(o); // 封存对象o, o对象将不能再添加属性
Object.getOwnPropertyDescriptor(o, "a");
// {value: {…}, writable: true, enumerable: true, configurable: false}
```

3. 冻结(freeze): 相当于调用`Object.seal(..)`并把所有已有属性标记为`writable: false`;

```javascript
var o1 = {a: obj}
Object.freeze(o1) // 冻结对象o1, 不能再改变已有属性值
Object.getOwnPropertyDescriptor(o1, "a")
{value: {…}, writable: false, enumerable: true, configurable: false}
```

对`o1.a`改变值是无效的, 但a引用的对象obj不会受到影响

### 枚举和迭代
`obj.propertyIsEnumerable(propName)` 方法检查属性是否直接存在于该对象且可枚举;

```javascript
var obj1 = {a: 3, b: 2};
Object.defineProperty(
	obj1,
"a",
	{ enumerable: true}
);
Object.defineProperty(
	obj1,
	"b",
	{ enumerable: false}
);
obj1.propertyIsEnumerable( "a" ); // true
obj1.propertyIsEnumerable( "b" ); // false
```

`in`操作符包含对象及其原型链上所有属性;

```javascript
"a" in obj1 && "b" in obj1 && "toString" in obj1 // true 
```

`Object.keys(..)`返回对象上所有可枚举属性的数组;

```javascript
Object.keys(obj1) // ["a"]
```

`Object.getOwnPropertyNames(..)` 返回对象上所有属性(不论是否可枚举)的数组

```javascript
Object.getOwnPropertyNames(obj1) // ["a", "b"]
```

`for...in`循环迭代对象及其原型链上所有可枚举属性

```javascript
var obj2 = Object.create(obj1);
Object.keys(obj2)  // []
for (var p in obj2) {
	console.log(p); // "a", 为从obj1继承的可枚举属性
}
```