---
layout: post
title: 复制内容到剪贴板
---

## 复制内容到剪贴板

JavaScript本身具有利用剪贴板功能的API，但需要是可以选择文本的区域；`document.execCommand`是与富文本编辑器交互的主要方式，接收的三个参数主要为

+ 要执行的命令名称(string)
+ 是否为当前命令提供用户界面(boolean),为保证最好的兼容性，应始终设置为`false`
+ 执行命令必须的值(如不需传值则传递`null`)
其中利用剪贴板操作的命令分别是 `copy`, `cut`, `paste`, 使用它们对执行时当前选择的文本有效，不需要第三个参数。

```
// 如果不希望对显示的文本进行编辑，可以在`<textarea>`标签中添加`readonly`属性
<div id="box">
      <textarea id="text" readonly>我是你要复制的值,readonly</textarea>
      <button id="copy">复制</button>
      <button id="silentcopy">隐性复制</button>
      <textarea id="text1"></textarea>
</div>
<script>
      const doc = document;
      const copyBtn = doc.querySelector('#copy');
      const silentcopyBtn = doc.querySelector('#silentcopy');
      copyBtn.addEventListener('click',function(){
        const text = doc.querySelector('#text');
        text.select();
        const dupliment = doc.execCommand('copy',false);
      })
      //不显示选择文本的效果
      silentcopyBtn.addEventListener('click',function(){
        const text = doc.querySelector('#text');
        const currentFocus = doc.activeElement;
        // text.focus();
        // text.setSelectionRange(0,text.value.length);
        text.select();
        doc.execCommand('copy',false,null);
        alert('copy successfully');
        currentFocus.focus();
      })
</script>
```

实现取消选中可以使用 `element.setSelectionRange(0,0)`或`element.blur()`使当前选中的元素失去焦点，也可以取消选中。