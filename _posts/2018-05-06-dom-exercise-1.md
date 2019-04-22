---
layout: post
title: DOM练习小记--一个简单的Web页面游戏
---
资料源自《Head first Ajax》 第六章（小时候我也有这么一个游戏板，图片是米奇……）

这是一个基于DOM的二维拼图游戏，HTML和CSS文档是原书的文档，我只是在initial版本中（没有js文件）添加了自己的js文件。
游戏规则是点击空白格上、下、左、右四个方向相邻的图片，会把该图片移动到空白格中，然后继续，直到把所有图片的位置按顺序排列，最后一个为空格，就算赢了。

文档结构并不复杂，静态页面是这个样子：

![initial_page](/images/init_page_6.JPG)

在`id="puzzleGrid"`的`div`中建立一个4row*4col的`table`，图片都放在对应的单元格中；每个单元格的id设置为"cell+行号+列号"；每个图片的`alt`属性也和图片本身显示的数字相同；点击事件注册在表格单元格上。

这里贴上第一行表格的HTML代码，其他行以此为参照：
```html
   <tr>
     <td id="cell11">
      <img src="images/07.png" alt="7" height="69" width="69">
     </td>
     <td id="cell12">
      <img src="images/06.png" alt="6" height="69" width="69">
     </td>
     <td id="cell13">
      <img src="images/14.png" alt="14" height="69" width="69">
     </td>
     <td id="cell14">
      <img src="images/11.png" alt="11" height="69" width="69">
     </td>
    </tr>
```

## 基本的思路：

1. `swapTiles(selectedCell, destinationCell)`: 主要动作在于交换图片，接收两个参数，点击的单元格和目标单元格（空格）；交换图片后判断是否赢得游戏；
2. `initPage`: 页面加载完成后初始化页面，先得到所有单元格列表，遍历每个单元格注册点击事件函数 `tileClick`;
3. `tileClick`: 每次点击一个单元格，判断是否为空（`isEmpty(cell)`）,如果是空格则不响应；如果非空格，则得到图片的位置标记并得到上、下、左、右的单元格，找到其中是空格的单元格，与之交换图片，都是非空格则不响应；
4. `isEmpty(cell)`: 接收单元格为参数，判断单元格内图片是否为空图片，返回`true/false`；
5. `getCell(pos)`: 接收单元格位置为参数，返回对应单元格，如找不到则返回`false`;
6. `isCompleted`: 遍历图片列表，根据图片顺序判断是否完成游戏，返回`true/false`；
7. 再加一个增加window.onload事件函数的方法`addLoadEvent(func)`;

没有按照书上的来，先自己尝试写了下，顺便巩固一下DOM的知识。代码如下：

1. 
```javascript
function swapTiles(selectCell, destinationCell) {
  //不能直接用firstChild，是回车；
  var selectImage = selectCell.getElementsByTagName("img")[0];
  var destinationImage = destinationCell.getElementsByTagName("img")[0];
  selectCell.replaceChild(destinationImage,selectImage);
  destinationCell.appendChild(selectImage);
  // 检测游戏结果
  if (!isCompleted()) {
    document.getElementById("puzzleGrid").className = "win";
  }
}
```
2. 
```javascript
// 初始化页面，找到对应表格并给每个表格绑定点击事件处理函数tileClick;
function initPage() {
  var puzzleGrid = document.getElementById("puzzleGrid");
  var cells = puzzleGrid.getElementsByTagName("td");
  var i, len = cells.length;
  for (i=0; i < len; i++) {
    var cell = cells[i];
    cell.onclick = tileClick;
  }
}
```
3. 
```javascript
function tileClick() {
  if (isEmpty(this)) {
    return false;
  }
  var pos = +(this.id.substr(4));
  var posUpCell = getCell(pos-10), 
      posDownCell = getCell(pos+10), 
      posLeftCell = getCell(pos-1), 
      posRightCell = getCell(pos+1),
      destinationCell; 
  switch(true) {
    case isEmpty(posUpCell):
    destinationCell = posUpCell;
    break;
    case isEmpty(posDownCell):
    destinationCell = posDownCell;
    break;
    case isEmpty(posLeftCell):
    destinationCell = posLeftCell;
    break;
    case isEmpty(posRightCell):
    destinationCell = posRightCell;
    break;
    default: return false;
  }
  swapTiles(this, destinationCell);
}
```
4. 
```javascript
function isEmpty(cell) {
  if (!cell) return false;
  var cellimg = cell.getElementsByTagName("img")[0];
  if (cellimg.alt === "empty") {
    return true;
  } else {
    return false;
  }
}
```
5. 
```javascript
function getCell(posNum) {
  var cellid = "cell"+posNum;
  if (!document.getElementById(cellid)) {
    return false;
  } else {
    return document.getElementById(cellid);
  }
}
```
6. 
```javascript
// 判断是否赢得游戏
function isCompleted() {
  // 用数组保存每个图片的alt值
  var arr = [],
      div = document.getElementById("puzzleGrid"),
      imgs = div.getElementsByTagName("img"),
      len = imgs.length,
      i;
  for (i=0; i < len; i++) {
    var img = imgs[i],
        imgAlt = img.alt;
        arr.push(imgAlt);
  }
  if (arr.join("") === "123456789101112131415") {
    return true;
  } else {
    return false;
  }
}
```
7. 
```javascript
//添加window.onload事件函数；
function addLoadEvent(func) {
  var oldload = window.onload;
  if (typeof oldload !== "function") {
    window.onload = func;
  } else {
    window.onload = function () {
      oldload();
      func();
    }
  }
}
addLoadEvent(initPage);
```

## 总结几点经验：

1. 一个父节点中除了我们想要的元素节点之外，可能浏览器查询到的也包含“回车符”、“空格”这类文本节点，所以`node.firstChild/lastChild`也许只是回车符，用这个方法获取节点时一定还要再加一层判断，`if (node.firstChild.nodeType === 1)`, 如果不是，就继续找`nextSibling`.
在这里我直接用了`div.getElementByTagName("img")[0]`;相比之下也许获取`nodeList`的方式更容易造成性能问题，DOM扩展中新增的`firstElementNode`这一类元素版可能会是更好的选择。

2. 第一次我的`getCell(pos)`和`isEmpty(pos)`两个函数都是接收单元格位置标记`pos`作为参数，它们都需要先得到对应的单元格，这样造成了部分重复代码，所以进一步分离，把先把`pos`传递给`getCell(pos)`得到单元格，再把它的结果传给`isEmpty(cell)`就可以达到相同的效果了。而且`isEmpty(cell)`还可以接收单元格（比如被点击的"`this`"）进行判断；

3. 在循环遍历得到累加字符串时，和书中不同，我采用数组`arr=[]`来保存每一项内容并最后用`arr.join("")`的方式输出；不过根据从网上其他资料了解到的，其实这种方法在现代浏览器（其他及IE6+）不比字符串拼接更高效。（[相关链接参考](https://www.zhihu.com/question/19747496)）

***
2018-5-6

