---
layout: post
title: react-art 初步
---

## react-art 初步
引入：
1. 下载依赖：`yarn add react-art`
2. 在文件中引入： `import ReactArt from 'react-art';`
3. 声明相应组件： `const {Shape,Transform,Surface,Text} = ReactART;`
4. 如果想要使用SVG模式绘图，需要添加引入：`import 'art/modes/svg'`,同时如果希望在SVG模式下使用path类(实例可作为shape的d属性)，需要引入SVGPath: `import SVGPath form 'art/modes/svg/path'`;

以下以**web端**、**使用SVG模式**为例。
```
import React, { Component } from 'react';
import ReactART from 'react-art';
import 'art/modes/svg';                  // SVG模式，不引入时可能是默认canvas模式
import SVGPath from 'art/modes/svg/path' // SVG模式下；canvas模式下可以直接使用ReactART.Path;

const {Surface, Text, Shape, Transform} = ReactART;
```
1. **Surface**是绘图容器，就像画布，常用属性如：
    + `width`,`height`：定义绘图区宽和高，
    + `style`：定义`margin`,`padding`,`background-color`等格式；
2. **Shape**是主要用来绘图的组件，就像一支画笔，可以自定义它的笔记颜色/粗细；绘图路径通过传入`d`属性([参考MDN释义](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute/d/))来定义；Shape常用属性如：
    + `d`: 绘图笔路径，可以是字符串形式也可以是Path实例；
    + `stroke`: 画笔颜色；
    + `fill`: 填充颜色；
    + `strokeWidth`: 画笔宽度；
    + `transform`: 需传入Transform的实例，定义图形的移动/旋转等命令；
3. **Text** 是放置文字的组件，可以直接在闭合标签中添加文字，但同样需要定义组件的坐标/字迹格式等才能正确显示；目前没有找到自动换行的方法，也不能定义宽高属性，所以使用时默认为单行；常用属性如：
    + `x`, `y`: 分别定义该文本组件的横坐标和纵坐标，代表组件左上角在画布上的位置；
    + `font`: 字体属性，按照“font-weight font-size font-family”定义，如`font="normal 14px Microsoft Yahei"`
    + `strokeWidth`, `stroke`, `fill`, `transform` 等属性用法与Shape相同，`stroke`指的是文字边缘颜色，`fill`指文字填充颜色，不设置`fill`属性时为空心字体。
4. **Transform** 是一个类，例如可以通过`const transform = new Transform().rotate(90)`创建一个实例，传入shape或Text等组件中的`transform`属性，即可使其逆时针旋转90°；常用命令如：
    + `translate(x,y)`: 相对当前位置位移；
    + `scale(x,y)`: 缩放，宽度方向x倍，长度方向y倍，只传一个参数时，宽度和长度方向都扩大相同倍数；
    + `move(x,y)`,`moveTo(X,Y)`: 分别为按相对/绝对坐标移动；
    + `rotate(deg, x, y)`: 旋转角度deg, x和y为旋转的原点; 如果不传入原点参数，Shape图形默认为绝对坐标(0,0)，对Text组件则默认为Text组件的原点。
5. **SVGPath** 是SVG模式下的Path类，可以通过`const path = new SVGPath().moveTo(100,100)...`的方式创建一个路径实例，将其传入Shape组件的`d`属性，即可进行对应路径的绘制；常用的命令如下：
    + `moveTo(X,Y)` 移动画笔至绝对坐标(X,Y)；
    + `move(x,y)` 相对当前位置移动相对距离(x,y)；
    + `lineTo(X,Y)` 向绝对坐标(X,Y)画直线；
    + `line(x,y)` 向相对当前位置距离(x,y)的点画直线；
    + `arc(x,y,rx,ry,LargeArcFlag)` 以相对当前位置(x,y)距离的点为目标，以(rx,ry)为长短半径，顺时针画椭圆曲线, LargeArcFlag指定画大弧(1)还是小弧(0)；
    + `arcTo(X,Y,rx,ry,LargeArcFlag)` 与arc相似，目的坐标为绝对坐标(X,Y);
    + `counterArc(x,y,rx,ry,LargeArcFlag)` arc的逆时针版本；
    + `counterArcTo(X,Y,rx,ry,LargeArcFlag)` arcTo的逆时针版本；
    + `curve()` 贝塞尔曲线，可接受2个/4个/6个参数;([参考资料](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute/d#Curveto/))
    + `close()` 从当前点向当前路径起始点画闭合直线；(每次画笔通过`move`,`moveTo`移动后都可以看作是新起点)

例：
```
// 在react的组件中使用
const path = new SVGPath()
    .moveTo(200,100)
    .counterArcTo(300,200,30,10,0)
    .line(-100,100)
    .arc(100,100,30,10,0)
    .close(); 

// ……
<Surface 
    width={350} 
    height={350} 
    style={{
        margin:"50px", 
        backgroundColor:"#eef",
    }}
>
    <Shape 
        d={path} 
        stroke={"#55e"} 
        strokeWidth={3}
        fill="#ccf"
    />
    <Text
        x={200}
        y={200}
        strokeWidth={1}
        fill="#eaa"
        font="bold 20px Microsoft Yahei"
        transform={new Transform().rotate(-45)}
    >
    这是一个例子
    </Text>
</Surface>
```
![example](/images/art-exp.png)

[参考demo](https://github.com/AnneBai/simple-art-svg-demo)

——————————