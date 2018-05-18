---
layout: post
title: DOM练习小记--简单的拼单词游戏
---
（资料源自《Head First Ajax》第七章）

书中另一个单页游戏案例，综合了DOM和事件处理，先总结一下页面布局和js编程的思路。涉及到服务端交互的暂时按下，毕竟还没有自己把服务端环境搭起来。

## 1) 内容简介

书中给的样本页面和样式是布置好的，初始静态页面如下：

![初始页面](/images/init_page_7.JPG)

1. 4row*4col布局，每次初始化都会随机出现16个字母图片；
2. 点击任意字母会把对应字母输入到旁边字母框中，每次提交前每个字母只能点击一次；
3. 需要拼出一个合法的单词，OK后点击submit提交（Ajax）；
4. 如果合法则在分数框计分数(元音+2，辅音+1)，并把该单词输出在下面的单词框中；如果不合法则提示单词错误；
5. 输入框和字母盘刷新；

### HTML/CSS

与拼图游戏的表格布局不同，这一次的布局使用了“`<div>`+CSS”，相比表格布局，CSS布局的优点是更灵活，更保持在所有浏览器中的一致性，并有利于优化和维护；
```
  <div id="letterbox">
   <a href="#" class="tile t11"></a>        
   <a href="#" class="tile t12"></a>        
   <a href="#" class="tile t13"></a>        
   <a href="#" class="tile t14"></a>
   ……
```
字母盘大概的布局像这样重复的4列，通过`class`的通用类名和特殊类名能方便地设置样式和定位；

对于每个方块中的字母图片也是在CSS中设置的背景：
```
#letterbox a.tile { background: url('../images/tiles.png') ……}
#letterbox a.la { background-position: 0px 0px; }
#letterbox a.lb { background-position: -80px 0px; }
#letterbox a.lc { background-position: -160px 0px; }
#letterbox a.ld { background-position: -240px 0px; }
#letterbox a.le { background-position: -320px 0px; }
#letterbox a.lf { background-position: -400px 0px; }
……
```
实际上为所有方块中的背景加载的背景是同一张图，只是通过为不同字母标志的类名设置不同的显示位置，就能显示对应的字母图像，而不必每次都为每个方块去加载单个的图像。

旁边的输入框和单词收集框都是`div`，有各自`id`，在操作DOM增删节点时非常方便。

## 2) 总体思路
1. 通用方法：除需要添加`window.onload`事件的`addLoadEvent`和创建请求的`createReqest`之外，因为涉及到一个元素同时拥有多个类名，还需要一个在原有类名基础上增加类名的`addClassName`方法。
2. `initPage`, 初始化页面时，字母盘中随机生成16个字母，需要随机生成数字并输出对应字母的方法`randomizeTiles`; 并为每个字母（链接）和`submit`框绑定点击事件处理函数；
3. `randomizeTiles`, 关于随机字母，书中的客户给出了26个字母在100个字母中出现的频率表，可以按照这个表初始化一个数组，然后用0~99之间的随机数（`Math.floor(Math.random()*100`)）作为索引值从中取出对应字母；
4. `addLetter`, 点击字母链接处理函数：首先找出所点击图片对应的字母；把该字母输入到输入框中，并禁用已点击过的图片链接；
5. `submitWord`, 点击提交框处理函数：确认输入框不为空，将其内容发送请求到服务器确认是否合法，并设置回调函数`updateScore`处理服务器返回的响应；
6. `updateScore`, 处理接收到的响应：确认是否合法，计算分数，把合法结果附加到单词收集框并清空输入框，最后刷新字母盘；

## 3) 代码要点
1. `addClassName`通用函数

   最初是只接收元素和类名两个参数，直接在原来的基础上添加，但后来发现这样会无休止地叠加下去，所以有必要把上次添加的相同功能的类名删除掉再重新添加。每次刷新字母盘，上次刷新添加的字母类名，以及点击之后添加的`disabled`类都可以删掉。所以每次添加类名前只需要保留前两个类名就可以了。我给`addClassName`增加了第三个可选参数`number`, 即添加前保留原类名的数量。
```
function addClassName(element,name,number) {
  var oldclass;
  if (typeof element.className !== "string") {
    oldclass = "";
  } else if (number !== undefined) { //未指定number;
    var classArr = element.className.split(/\s+/);
    classArr.length = number;
    oldclass = classArr.join(" ");
  } else {
    oldclass = element.className;
  }
  element.className = oldclass+" "+name;
}
```
2. `initPage`初始化页面
```
addLoadEvent(initPage); //页面加载完成后执行
function initPage() {
  var letterbox = document.getElementById("letterbox");
  var letterlinks = letterbox.getElementsByTagName("a");
  var i, len = letterlinks.length;
  for (i=0; i < len; i++) {
    var letterlink = letterlinks[i];
    randomizeTiles(letterlink);
    letterlink.onclick = addLetter;
  }
  var submitbtn = document.getElementById("submit");
  submitbtn.onclick = submitWord;
}
```
在这里因为生成随机字母和绑定点击事件都需要遍历所有`<a>`元素，所以我让`randomizeTiles`接收元素作为参数而把遍历放在这个函数中，一次完成两件工作，然后再为提交按钮绑定事件。但是后来发现，当点击提交按钮之后，字母表盘需要刷新（并解除禁用重新绑定）而提交按钮不需要再绑定，所以需要把字母盘和提交按钮的点击事件分开绑定，好单独调用刷新字母盘的方法。最后我把刷新字母盘的工作都交给了`randomizeTiles`.


更改后
```
addLoadEvent(initPage);
function initPage() {
  randomizeTiles();
  var submitbtn = document.getElementById("submit");
  submitbtn.onclick = submitWord;
}
```
3. `randomizeTiles`生成随机字母盘
```
function randomizeTiles() {
  var frequencyTable= new Array("a", "a", "a", "a", "a", "a", "a", "a", "b", "c", "c", "c", "d", "d", "d",
  "e", "e", "e", "e", "e", "e", "e", "e", "e", "e", "e", "e", "f", "f", "g",
  "g", "h", "h", "h", "h", "h", "h", "i", "i", "i", "i", "i", "i", "i", "j",
  "k", "l", "l", "l", "l", "m", "m", "n", "n", "n", "n", "n", "n", "o", "o",
  "o", "o", "o", "o", "o", "o", "p", "p", "q", "q", "q", "q", "q", "q", "r",
  "r", "r", "r", "r", "r", "s", "s", "s", "s", "s", "s", "s", "s", "t", "t",
  "t", "u", "u", "v", "v", "w", "x", "y", "y", "z");
  var letterbox = document.getElementById("letterbox"),
      letterlinks = letterbox.getElementsByTagName("a"),
      i, 
      len = letterlinks.length;
  for (i=0; i < len; i++) {
    var letterlink = letterlinks[i];
    var x = Math.floor(Math.random()*100);
    var classLetter = "l"+frequencyTable[x];
    addClassName(letterlink, classLetter, 2); 
    letterlink.onclick = addLetter;
  } 
}
```
其中数组frequencyTable是按照每个字母在100个字母中出现的频数（“客户”已提供）直接以字面量的形式创建。这样对运行来说可能是最高效的。反而最开始我思考要用什么方法通过循环复制生成数组更像是舍近求远的做法。如果输入费劲可以借助excel快速复制再整体复制文本过来，也会很方便。
4. `addLetter` 点击字母的事件处理
```
function addLetter() {
  var currentWord = document.getElementById("currentWord"),
      p;
  if (currentWord.childNodes.length === 0) {
    p = document.createElement("p");
    currentWord.appendChild(p);
  } else {
    p = currentWord.firstChild;
  }
  var letter = this.className.slice(-1),
      letterText = document.createTextNode(letter);
  p.appendChild(letterText);
  //禁用该图片
  this.onclick = null;
  addClassName(this, "disabled"); //用于改变图片样式
}
```
和书中不同的是，我觉得只在第一次输入字母的时候创建`<p>`子元素节点就可以了，后面都可以只在`<p>`中增加文本节点。 相对来说代码更简单一点。

5. `submitWord` 点击提交
```
function submitWord() {
  var currentWord = document.getElementById("currentWord");
  //如果为空则不提交
  if (currentWord.innerHTML == "" || currentWord.firstChild.innerHTML == "") {
    return false;
  } else {
    wordRequest = createRequest();
    if (wordRequest === null) {
      alert("Unable to create a request, sorry.")
      return false;
    } else {
      var url = "lookup-word.php?word="+currentWord.firstChild.innerHTML;
      wordRequest.onreadystatechange = updateScore;
      wordRequest.open("GET",url,true);
      wordRequest.send();
    }
  }
}
```
6. `updateScore` 处理响应
```
function updateScore() {
  if (wordRequest.readyState == 4) {
    if (wordRequest.status == 0) {
      var currentWord = document.getElementById("currentWord"),
          resText = wordRequest.responseText;
      if (resText === "-1") {
        currentWord.innerHTML = "";
        alert("It\'s not a proper word, please try again");
        randomizeTiles();
      } else if (!isNaN(resText)) {
        var wordList = document.getElementById("wordList"),
            p = currentWord.firstChild.cloneNode(true);
        wordList.appendChild(p);
        currentWord.firstChild.innerHTML = "";
        var score = document.getElementById("score"),
            oldscore = score.firstChild.nodeValue.slice(7);
            newscore = +oldscore + (+resText),
            scoreText = document.createTextNode("Score: "+newscore);
        score.replaceChild(scoreText, score.firstChild)
        randomizeTiles();
      } else {
        alert("Sorry, there is an error in the response.");
      }
    }
  }
}
```
这里先判断结果是否是-1（单词不合法），然后再判断是否是一个数值字符串（正常情况下只有这两种情况）;`isNaN()`这个方法是会尝试对参数进行数值转换，如果失败则会返回true，表明参数不是可转化为数值的类型。另外这里我原先用`innerHTML`改变元素内容，后也采用操作节点的方式进行了修改。

## 总结
1. 用到的DOM属性和方法：
+ `childNodes`
+ `appendChild()`
+ `replaceChild()`
+ `firstChild()`
+ `getElementById()`
+ `getElementsByTagName()`
+ `createElement()`
+ `createTextNode()`
2. 数组和字符串的方法：
+ `string.slice(index)`, 字符串从`index`位置向后截取子字符串， 如果有第二个参数则是到哪个位置停止截取；
  
  相似的还有`substring()`, `substr()`(第二个参数指定返回字符串个数)，它们接收负值参数的表现不同，`slice()`可接受两个负值参数;
+ `string.split()`, 把字符串基于指定的分隔符分割成若干子字符串返回它们组成的数组, 并可以指定返回数组的长度（第二个参数）；这个在`addClassName`中用于把原来的`className`分成子类名并利用指定`array.length`保留一定数量的类名；


这样我可以把
```
    var classArr = element.className.split(/\s+/);
    classArr.length = number;
```
改成

```
    var classArr = element.className.split(/\s+/, number);
```
  顺便复习下长得比较像的`splice()`方法，这是数组方法，接收的参数为(要删除的第一项位置， 要删除的项数，从该位置要插入的项……)；借此可以完成对数组任意项的删除、替换以及插入任意项；返回它删除的数组（如无则空数组）；值得注意的是，它操作的是原始数组。

  这样上面的代码还可以改成一步到位的：
  ```
    var classArr = element.className.split(/\s+/);
    classArr.splice(number, 2, name); //可能还有"disabled"类名所以删除2项；
  ```
  不过这里因为涉及到没有指定number的情况，所以没有改用这种方式。

3. 最后，是编程的过程中，尽量减少重复的DOM操作，一次查询或遍历最好能把一串不冲突的动作都完成。涉及到累加( `+=`, `push`等相关) 要记得看是替换还是不停累加下去；

***
2018-05-08


