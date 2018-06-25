---
layout: post
title: ES6随笔--基本数据类型的扩展
---

## 字符串
### Unicode表示法
`\uxxxx`表示的Unicode如果码数超过了0xFFFF的范围，只要放进大括号内就能识别；
    
### codePointAt()
用`codePointAt()`可以准确识别码数超出0xFFFF范围的Unicode字符；
    
`for...of`循环能够正确识别32位的UTF-16字符；
### String.fromCodePoint()
与`codePointAt()`作用刚好相反；根据传入的Unicede码数返回对应的字符；
### 遍历
`for...of`循环可以循环遍历字符串，并且可以识别大于`0xFFFF`的码点；
### at()
补充`charAt()`不能识别大于`0xFFFF`的码点的字符的缺陷；
### normalize
用于将Unicode合成字符正规化为JS可识别字符；
### includes(), startsWith(), endsWith()
补充`indexOf()`；均返回布尔值；
    
`includes()`、`startsWith()`分别是参数字符串是否在原字符串包含、是否在原字符串头部；第二个参数指从从第`n`个字符开始向后查找；
    
`endsWith()`表示参数字符串是否在原字符串的尾部，第二个参数是前`n`个字符；
### repeat()
接收数值参数，返回将原字符串重复`n`次的新字符串；参数为`Infinity`和复数会报错，参数为0~1小数、`NaN`会当做0；参数为小数会向下取整；参数为字符串会先转换为数值；
### padStart(), padEnd()
分别用于头部补全和尾部补全，接收两个参数：第一个参数指定字符串最小长度，第二个参数指定用于补全的字符串；
    
可用于补全数值位数，或者提示输入字符串格式；
```
    '123456'.padStart(10, '0') // "0000123456"
    '09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```
### matchAll()
返回一个正则表达式在当前字符串的所有匹配；
### 模板字符串
用反引号标识，可以当做普通字符串，也可以定义多行字符串（空格和缩进都会被保留）和在字符串中嵌入变量（使用`${}`,大括号中可以使用表达式、函数等）；
    
变量必须是已声明的变量；
    
### 标签模板
模板字符串被用来跟在函数名后，该函数会被调用来处理该字符串；如果模板字符串里有变量，会先将模板字符串处理成多个参数，再调用函数；处理成参数时，模板字符串中没有变量替换的基本字符串会组合为一个数组作为函数的第一个参数，处理的变量则向后依次排列；
```
      let a = 5;
      let b = 10;
      function tag(s, v1, v2) {
        console.log(s[0]);  //"hello"
        console.log(s[1]);  //"world"
        console.log(s[2]);  //""
        console.log(s[3]);  //undefined
        console.log(s);  //["hello ", " world ", "", raw: Array(3)]
        console.log(v1); // 15
        console.log(v2); // 50
        return "ok";
      }
      tag`hello ${a+b} world ${a*b}`;
```
标签模板可以用来过滤HTML字符串，防止用户输入恶意内容；
### String.raw
可用于处理模板字符串，将其中的变量替换，并对斜杠进行转义，方便后面取得原始字符串；
```
    console.log(String.raw`hello \n world ${a*b}`);  // hello \n world 50
    console.log`hello \n world ${a*b}`;  //  ["hello ↵ world ", "", raw: Array(2)] 50
```
## 正则表达式
### RegExp构造函数
第二个参数可以覆盖第一个参数为正则表达式时添加的flag;
### 字符串的正则方法
字符串可以使用正则表达式的方法`match()`/`search()`/`replace()`/`split()`，在实现时都调用`RegExp`的实例方法，使所有与正则相关的方法都定义在`RegExp`对象上；
### u修饰符
用于正确处理四个字节的UTF-16编码；
```
/^\uD83D/u.test('\uD83D\uDC2A') // false
```
使用大括号表示Unicede字符时，必须使用`u`修饰符；
### y修饰符
与`g`修饰符相近，但需要“粘连”，即下一个匹配项必须从上一个匹配项的下一个字符开始；
### `sticky`属性
布尔值--是否设置了`y`修饰符；
### `flags`属性
返回正则表达式的修饰符；
### s修饰符
行终止符：

+ U+000A 换行符（\n）
+ U+000D 回车符（\r）
+ U+2028 行分隔符（line separator）
+ U+2029 段分隔符（paragraph separator）
`s`修饰符用于匹配任意单个字符；
    
`dotAll`属性返回布尔值--是否处于dotAll模式（点代表所有字符）；

### 后行断言
    `x`只有在`y`后面才匹配；
```
//只匹配美元符号后面的数字
    /(?<=\$)\d+/.exec('Benjamin Franklin is on the $100 bill')  // ["100"]
//只匹配不在美元符号后面的数字
    /(?<!\$)\d+/.exec('it’s is worth about €90')                // ["90"]
```
后行断言的组匹配（从右向左贪婪模式的影响）和反斜杠引用与先行断言有所区别；由于是先匹配前面的字符，然后从右向左完成反斜杠引用；
### `\p{}`属性类写法
允许正则表达式匹配符合Unicode某种属性的所有字符；`\P{}`是匹配不符合Unicode某种属性的字符；

只对Unicode有效，使用的时候需要加上`u`修饰符;
`\p{Number}`可以匹配罗马数字；
### 具名组匹配
可以为每一个匹配组指定一个名字；写在尖括号里，放在圆括号内开头并加`?`前缀；
```
const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;

const matchObj = RE_DATE.exec('1999-12-31');
const year = matchObj.groups.year; // 1999
const month = matchObj.groups.month; // 12
const day = matchObj.groups.day; // 31
```
### 解构变量赋值和替换
具名组匹配与解构变量赋值结合可以直接为变量赋值：
```
let {groups: {one, two}} = /^(?<one>.*):(?<two>.*)$/u.exec('foo:bar');
one  // foo
two  // bar   
``` 
字符串替换时，引用具名组使用`$<组名>`的写法；
    
在正则表达式内部引用具名组，用`\k<组名>`的写法； 
### `String.prototype.matchAll()`方法
可以把所有匹配项取出，返回匹配项的遍历器，把遍历器转为数组可以使用`...`操作符或`Array.from()`方法；
```
const string = 'test1test2test3';

// g 修饰符加不加都可以
const regex = /t(e)(st(\d?))/g;

for (const match of string.matchAll(regex)) {
  console.log(match);
}
// ["test1", "e", "st1", "1", index: 0, input: "test1test2test3"]
// ["test2", "e", "st2", "2", index: 5, input: "test1test2test3"]
// ["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]
```
转为数组：
```
// 转为数组方法一
[...string.matchAll(regex)]

// 转为数组方法二
Array.from(string.matchAll(regex));
```

***
2018-5-17