---
layout: post
title: CSS--关于line-height
---

`line-height`对于文本排版来说很有用，有时候也可以用来垂直居中；但如果使用不当也会出现意料之外的效果。之前对它只是简单地设置一个带单位的高度，发现有时候子元素的文字结果并不像自己想的那样。查询了一下，才了解原来这里有一个“计算”。

`line-hight`字面理解就是“行高”，它可以应用在所有元素上而不只是块级元素（初学者的谬误）。它可以被赋予一个数值、一个带单位的高度或者一个百分数，默认时是关键字`normal`；

## 默认情况
例如下图为chrome浏览器中未设置`line-height`时的呈现(font-size为默认)，

![normal](images/line-height-1-0.png)

代码如下：
```
    <div>
        <h2>Far out in the uncharted backwaters of the unfashionable</h2> 
        <p>end of the western spiral arm of the Galaxy lies a small unregarded yellow sun.</p>
    </div>

```
即使是在浏览器中把页面缩小至50%，或放大到150%，看起来还是很自然的：

![normal](images/line-height-1-1.png)

![normal](images/line-height-1-2.png)

## 设置`line-height`
但是如果用不同的方式设置了`line-height`会怎样呢？

```
    .div1 { line-height: 1.2; }   /* number */ 
    .div2 { line-height: 1.2em;}   /* length */ 
    .div3 { line-height: 120%; }   /* percentage */
    
```

![line-height in different value](images/line-height-2-0.png)


当设置为数值1.2, `line-height`实际在应用时不同字体的行高是字体实际的大小的1.2倍；对于`div`中的`h2`元素，实际字体比普通字体要大，它的行高也会是它本身字体大小的1.2倍(16px\*1.5\*1.2=28.8px)，`p`元素内的文字则按行高(16px\*1.2=19.2px)呈现，所以看起来也还是比较自然；

但是如果父元素的`line-height`设置了1.2em或120%,那`h2`和`p`元素中的文字都会直接继承1.2em的绝对值(19.2px)或父元素设置/默认字体大小的120%，同样也是绝对值19.2px，所以对于字体较大的`h2`元素，文字看起来有些挤了。

需要注意的是，从百分比或带单位的长度继承的绝对值是父元素中应用的`line-height`高度，只与父元素本身的`font-size`和`line-height`设置有关，除非在子元素重新设置`line-height`，否则都会继承父元素中的绝对行高;

比如把`p`元素字体都设置`font-size:20px`，父元素行高为`normal`和`line-height:1.2`的情况下，实际行高根据字体的变化也调整到24px;而另外两种设置的`div`内所有子元素的行高还是19.2px：

![line-height in different value](images/line-height-2-1.png)

或者把`h2`的字体重设为30px, 设置`line-height: 1.2`的盒子里`h2`文字的行高变为了36px，而另外两个依然是19.2px；

![line-height in different value](images/line-height-2-2.png)


## 总结一下：
`line-height`是会继承的属性，但是并不是直接继承父元素的设置，而是会经过计算。对于带单位的高度或百分数定义的`line-height`，子元素的`line-height`直接继承其在父元素中的绝对值；而对于数值定义或`normal`，则会按指定值继承后在子元素中根据子元素内的字体大小重新计算，或者也可以认为是一种“相对继承”（个人说法，未必准确）。

尤其是对于文字排版，最好用不带单位的数值来指定`line-height`，这样能够保证在不同字体大小的情况下都能够等比例变化和显示，不容易出现意外。

参考文献：
    [MDN CSS line-height](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height)