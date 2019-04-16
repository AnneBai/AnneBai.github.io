---
layout: post
title: JS--相等性判断
---

### JS提供的相等性判断操作：
1. 抽象(非严格)相等：`==`，两侧数据类型相同时同严格相等，不同时会按规则进行类型转换后再比较；
2. 严格相等：`===`，不转换数据类型，当类型和值均相同认为全等，其中数值类型有2个特例：`NaN`不等于自身，`+0`等于`-0`；
3. 同值零相等(sameValueZero): 除了`NaN`等于其自身, 其他与严格相等一样;
4. 同值相等：`Object.is`, 除了对特例的处理不同：`NaN`与自身同值，`+0`与`-0`不同值（`Object.is(NaN,NaN)` --> true，`Object.is(+0,-0)` --> false）其他比较情况都与严格相等一样；

### 非严格相等的类型转换：
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
+ [Equality comparisons and sameness
](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness#Same-value-zero_equality)