---
layout: post
title: 类型转换
---

### ToString

非string值转换为string表现形式, 例如`(...).toString()`和`String(...)`方法. `undefined`和`null`值不能调用`toString()`, 但是`String()`方法可以接受任何参数.

```
// 普通对象默认返回内部[[class]]
({a: "a"}).toString()
"[object Object]"

// 数组返回所有值字符串化后通过","分割的字符串
[1,2,3].toString()
String([1,2,3]);
// 均返回 "1,2,3"

// 数值, undefined, null, symbol, 函数, 返回其本身字符串化的值
(1).toString()
// "1"  数值需包含在括号中, 否则会因小数点歧义而产生错误
String(null) // "null"
String(undefined) // "undefined"

(function a() {console.log(a)}).toString()
// "function a() {console.log(a)}"
```
#### JSON序列化

JSON 不安全的值包括: `undefined`、`function`、（ES6+）`symbol`、和带有循环引用的 `object`.

JSON.stringify(...)参数为`undefined`、`function`、`symbol`时会自动忽略, 如果它们在`array`中出现则会被替换为`null`, 如果它们在`object`中作为属性值出现, 则会被剔除.

```
JSON.stringify(Symbol()); // undefined
JSON.stringify(() => {})); // undefined
JSON.stringify(undefined); // undefined

JSON.stringify([1, () => {}, Symbol(), undefined, 5])
// "[1,null,null,null,5]"

JSON.stringify({a: undefined, b: Symbol(), c: () => {}, d: "d"})
// "{"d":"d"}"
```
带有循环引用的对象, 使用`JSON.stringify()`时会抛出错误:

```
var a = {c: "c"};
var b = {};
a.b = b;
b.a = a;
JSON.stringify(a);
// Uncaught TypeError: Converting circular structure to JSON
```

通过为对象自定义`toJSON()`方法去掉有循环引用的属性, 返回这个对象的安全版本, 可以避免报错:
```
a.toJSON = function () {
    return {
        c: this.c,
    };
};
JSON.stringify(a);
// "{"c":"c"}"
```

> `toJSON()` 应当被翻译为：“变为一个适用于字符串化的 JSON 安全的值”，而不是像许多开发者错误认为的那样，“变为一个 JSON 字符串”。

`JSON.stringify()`可以传入第二个参数和第三个参数, 分别称为"替换器"和"填充符".

替换器可以作为属性过滤器, 规定序列化(递归地, 所以会对每个层次的属性过滤)时保留或省略哪些属性, 可以传入要保留属性名的数组, 或接受键值对作为参数并在满足保留条件时返回对应值的函数.

```
var o = {
    a: "a",
    b: "b",
    c: {
            ca: "ca",
            cb: "cb",
    },
}

JSON.stringify(o, ["b", "c"]);
// "{"b":"b","c":{}}"
JSON.stringify(o, ["b", "c", "ca"])
// "{"b":"b","c":{"ca":"ca"}}"

```

填充符可以用来设置缩进格式;

```
JSON.stringify(o, ["b", "c", "ca"], "    ") // 4个空格
/*
"{
    "b": "b",
    "c": {
        "ca": "ca"
    }
}"
*/
```

> `JSON.stringify(..)` 并不直接是一种强制转换的形式, 但对于合法的基本类型值的序列化结果与通过`ToString`抽象操作规则强制转换为string值的方式相同。

### ToNumber

把非number值转换为数值类型.

如果是对象或数组, 会首先被转换为它们的基本类型值的等价物, 而后这个结果值根据`ToNumber`强制转换规则转换为number值.

转换为基本类型值会用到`ToPremitive`抽象操作, 首先如果有`valueOf()`方法可用且调用后返回结果为基本类型值则用这个值进行强制转换; 否则继续调用`toString()`方法(如果可用)以求得到基本类型值; 如果到现在没有得到基本类型值, 会抛出`TypeError`错误

明确进行强制转换可使用: `Number()`方法, 一元加操作符`+`, `parseInt()`, `parseFloat()`等全局方法;

```
Number([]); // 0
Number(null); // 0
Number("") // 0
Number(); // 0
Number(false); // 0

Number(undefined); // NaN
Numebr({}); // NaN

Number([1]); // 1
Number(["2"]); // 2
Number([1, 2]); // NaN

var a = {
    value: [3,4,5],
    valueOf() {
        return a.value.join("");
    }
}
Number(a); // 345
```

#### parseInt
`parseInt()`方法的第一个参数应该是一个字符串, 如果不是, 会根据强制转换规则将其转换为string类型. 不同的是, 强制转换含有非数值字符的字符串会直接得出`NaN`, 但如果开头部分是合法的数字字符串, `parseInt`依然会解析, 直到遇到第一个非数字字符为止.

```
parseInt(new String(3)); // 3
// 这里因为String(new String(3))结果为"3";

parseInt([1,2,3], 2); // 1
// 这里因为String([1,2,3])结果为"1,2,3", 按二进制解析到第一个逗号遇到非数值字符, 停止解析;

```

```
parseInt( 1/0, 19 ); // 18
/* 这里首先把第一个参数转换为字符串, "Infinity", 然后按照19进制解析, 可解析的字符为`0-9`和`a-i`(代表10~18); 第一个字符`i`解析为18, 第二个字符"n"不在可解析字符范围, 停止解析. 所以最后输出18;
*/

// 同理, 按24进制就解析为:
parseInt(1/0, 24)
// 151176378
// ==> 18 * (24 ** 5 + 24 ** 2 + 24 ** 0) + 23 * (24 ** 1 + 24 ** 4) + 15 * (24 ** 3)
```

### ToBoolean

把非boolean值转换为boolean值. 对于转换后为`false`的值, 称为"falsy值",反之为"truthy"值. falsy值列表如下:
```
undefined
null
false
0, +0, -0, NaN
""
```

除了以上列表中的值, 其他没有明确存在于falsy列表中的值都是truthy值.

### 相等性判断

#### JS提供的相等性判断操作：
1. 抽象(非严格)相等：`==`，两侧数据类型相同时同严格相等，不同时会按规则进行类型转换后再比较；
2. 严格相等：`===`，不转换数据类型，当类型和值均相同认为全等，其中数值类型有2个特例：NaN不等于自身，+0等于-0；
3. 同值相等：`Object.is`, 除了对特例的处理不同：NaN与自身同值，+0与-0不同值（Object.is(NaN,NaN):true，Object.is(+0,-0):false）其他比较情况都与严格相等一样；


#### 非严格相等的类型转换：
1. 当两侧数据类型相同，等同于严格比较；
2. undefined或null类型不进行转换，`undefined`与`null`相等，与其他任何类型值都不相等；
3. 当有一个操作数为number类型值, 则将另一个操作数强制转换为number类型, 然后对比;
4. 当有一个操作数为boolean类型的值, 先将其强制转换为数值类型, 然后对比;
5. 当有一个操作数是string类型或number类型, 另一个操作数是对象, 会先对对象使用`ToPremitive`抽象操作强制转换为基本类型值(调用`valueOf()`和`toString()`, 得到基本类型值为止.)
6. 当两个操作数都是对象时，会比较其内部引用，当且仅当他们的引用指向内存中的相同对象（区域）时才相等，即他们在内存中的引用地址相同。

```

[1] == "1"; // true 数组转换为基本类型值, 相当于 "1" === "1";
[1] == 1; // true 数组转换为数值, Number(ToPremitive([1])) --> Number("1"), 相当于 1 === 1;
[1] == true // true 先将true转换为数值1, 再如上比较;
"true" == true // false 先将true转换为数值1, 再把字符串"true"转换为数值得到NaN, 相当于NaN === 1;

对"true"强制转换为数值结果为NaN, 可以如下验证:
Number.isNaN(Number("true")); // true

var a = {
    valueOf() {
        return 5;
    }
}
a == "5"; // true;
// 先将对象a调用valueOf()得到基本类型值5; 再比较 5 == "5"; 把字符串转换为数值, 相当于5 === 5;

```

参考:
[You-Dont-Know-JS/类型与文法](https://github.com/getify/You-Dont-Know-JS/blob/1ed-zh-CN/types%20%26%20grammar/ch4.md)

[MDN-比较操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Equality)