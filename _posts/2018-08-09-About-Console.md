---
layout: post
title: 浏览器Console常用命令
---
1. 快捷键：`ctrl`+`shift`+`I` 或 `F12`;
2. 打印：`console.log()`,其中的参数可以为多个，将会以空格间隔进行打印；
3. 格式化打印：

格式说明符以`%`开头，后面添加：
+ `s`  字符串
+ `d` or `i`  整数
+ `f`  浮点值
+ `o`  可扩展的DOM元素
+ `O`  可扩展的JS对象
+ `c`  使用提供的CSS样式格式化输出
        console.log('%cThis is the best time','color:#a25354;font-size:20px;')

4. 鲜明提示自定义错误消息：`console.assert()`
5. 清除控制台的输出：`console.clear()`
6. 返回代码中调用函数的次数：`console.count()`
```javascript
function clickHandler(){
    console.count(`click handler called`)
}     
for (var i = 0; i < 3; i++) {
    clickHandler()
}
// 每次调用输出调用次数
Click handler called: 1
Click handler called: 2
Click handler called: 3
```
7. 打印对象，如果是HTML元素会打印元素的DOM表示：`console.dir()`
8. 打印对象，如果是HTML元素会打印元素的XML表示：`console.dirxml()`
9. 控制台报错：`console.error()`
10. 计算代码执行所需时间，两个方法应传递相同的label参数：`console.time(label)` + `console.timeEnd(label)`
11. 打印调用堆栈`console.trace()`