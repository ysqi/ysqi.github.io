
---
date: 2017-03-13T08:20:07+08:00
title: "go_js模型样式选择"
description: ""
disqus_identifier: 1489364407864042809
slug: "go_jsmo-xing-yang-shi-shua-ze"
source: "https://my.oschina.net/u/2391658/blog/856228"
tags: 
- gojs 
categories:
- 编程语言与开发
---

一、使用官方样式
================

样式（shapes）地址：[http://gojs.net/latest/samples/shapes.html](http://gojs.net/latest/samples/shapes.html)

![](https://static.yushuangqi.com/blog/2017/0313075858r11uxdecek0.gif)

使用方法：

             myDiagram.nodeTemplateMap.add("End",
                  $(go.Node, "Spot", nodeStyle(),
                    $(go.Panel, "Auto",
                      $(go.Shape, "Circle", // 在此处设置样式
                        { minSize: new go.Size(40, 40), fill: "#DC3C00", stroke: null }),
                      $(go.TextBlock, "End",
                        { font: "bold 11pt Courier New,Microsoft Yahei", stroke: lightText },
                        new go.Binding("text"))
                    ),
                    makePort("T", go.Spot.Top, false, true),
                    makePort("L", go.Spot.Left, false, true),
                    makePort("R", go.Spot.Right, false, true)
                  ));

二、自定义样式
==============

通过`Shape`你可以构建一个几何图形，并且可以控制它的形状和轮廓颜色以及填充色等。

图形
----

你可以通过`Shape.figure`属性设置它的形状。让你使用`GraphObject.make`方法构建时，你可以将形状参数以字符串形式作为第二个参数。你可能还需要通过`GraphObject.desiredSize`、`GraphObject.width`、`GraphObject.height`属性设置`Shape`的尺寸大小。

下面的例子中列出了一些常用的`Shape`形状，并且你可以看到它们的名字：

    diagram.add(
    $(go.Part, "Horizontal",
      $(go.Shape, "Rectangle",
                  { width: 40, height: 60, margin: 4, fill: null }),
      $(go.Shape, "RoundedRectangle",
                  { width: 40, height: 60, margin: 4, fill: null }),
      $(go.Shape, "Ellipse",
                  { width: 40, height: 60, margin: 4, fill: null }),
      $(go.Shape, "Triangle",
                  { width: 40, height: 60, margin: 4, fill: null }),
      $(go.Shape, "Diamond",
                  { width: 40, height: 60, margin: 4, fill: null })
    ));

结果：\

[![shape](https://static.yushuangqi.com/blog/2017/0313075859vqhcz1nmjol.jpg)](https://static.yushuangqi.com/blog/2017/0313075859vqhcz1nmjol.jpg)

填充色与轮廓
------------

可以通过`Shape.stroke`属性设置`Shape`的轮廓线条颜色。通过`Shape.strokeWidth`设置轮廓线条的粗细度。通过`Shape.fill`属性设置填充颜色。

    diagram.add(
    $(go.Part, "Horizontal",
      $(go.Shape, { figure: "Club", width: 40, height: 40, margin: 4
                    }),  // default fill and stroke are "black"
      $(go.Shape, { figure: "Club", width: 40, height: 40, margin: 4,
                    fill: "green" }),
      $(go.Shape, { figure: "Club", width: 40, height: 40, margin: 4,
                    fill: "green", stroke: null }),
      $(go.Shape, { figure: "Club", width: 40, height: 40, margin: 4,
                    fill: null, stroke: "green" }),
      $(go.Shape, { figure: "Club", width: 40, height: 40, margin: 4,
                    fill: null, stroke: "green", strokeWidth: 3 }),
      $(go.Shape, { figure: "Club", width: 40, height: 40, margin: 4,
                    fill: null, stroke: "green", strokeWidth: 6 }),
      $(go.Shape, { figure: "Club", width: 40, height: 40, margin: 4,
                    fill: "green", background: "orange" })
    ));

结果：\

[![shape](https://static.yushuangqi.com/blog/2017/0313075859o2cg2iarbsw.jpg)](https://static.yushuangqi.com/blog/2017/0313075859o2cg2iarbsw.jpg)

`Shape.stroke`和`Shape.fill`属性的值一般使用CSS设置，它们的默认颜色都是黑色。但是我们最常用的还是将它们设置为`null`或`transparent`，前者意思是空，后者是透明，在表象上，这两者是没有差距的，但是在点击行为上有区别。如果设置为`null`，那么只能点击图形的轮廓才能选中，点击图形里面是无法选中的。而如设置为`transparent`，点击图形的任何地方都可以选中该图形。

    diagram.add(
    $(go.Part, "Table",
      $(go.Shape, { row: 0, column: 0, figure: "Club", width: 40, height: 40, margin: 4,
                    fill: "green" }),
      $(go.TextBlock, "green", { row: 1, column: 0 }),
      $(go.Shape, { row: 0, column: 1, figure: "Club", width: 40, height: 40, margin: 4,
                    fill: "white" }),
      $(go.TextBlock, "white", { row: 1, column: 1 }),
      $(go.Shape, { row: 0, column: 2, figure: "Club", width: 40, height: 40, margin: 4,
                    fill: "transparent" }),
      $(go.TextBlock, "transparent", { row: 1, column: 2 }),
      $(go.Shape, { row: 0, column: 3, figure: "Club", width: 40, height: 40, margin: 4,
                    fill: null }),
      $(go.TextBlock, "null", { row: 1, column: 3 })
    ));

结果：\

[![shape](https://static.yushuangqi.com/blog/2017/0313075900idjnnt4cqqo.jpg)](https://static.yushuangqi.com/blog/2017/0313075900idjnnt4cqqo.jpg)

几何形状
--------

形状其实就是若干个不同坐标的点，然后被线连起来形成的。上述的例子中，我们使用的都是gojs定义好的一些形成，然而gojs也支持用户自定义形状，可以使用`Shape.geometry`属性。可以通过`Geometry.parse`来创建一个几何图形，不过`Geometry`的表达式交为复杂，我们将在后期对其进行详细的介绍。

下面的例子中，我们展示了通过`Geometry.parse`方法创建自定义“W”几何图形的过程：

    var W_geometry = go.Geometry.parse("M 0,0 L 10,50 20,10 30,50 40,0", false);
    diagram.add(
    $(go.Part, "Horizontal",
      $(go.Shape, { geometry: W_geometry, strokeWidth: 2 }),
      $(go.Shape, { geometry: W_geometry, stroke: "blue", strokeWidth: 10,
                    strokeJoin: "miter", strokeCap: "butt" }),
      $(go.Shape, { geometry: W_geometry, stroke: "blue", strokeWidth: 10,
                    strokeJoin: "miter", strokeCap: "round" }),
      $(go.Shape, { geometry: W_geometry, stroke: "blue", strokeWidth: 10,
                    strokeJoin: "miter", strokeCap: "square" }),
      $(go.Shape, { geometry: W_geometry, stroke: "green", strokeWidth: 10,
                    strokeJoin: "bevel", strokeCap: "butt" }),
      $(go.Shape, { geometry: W_geometry, stroke: "green", strokeWidth: 10,
                    strokeJoin: "bevel", strokeCap: "round" }),
      $(go.Shape, { geometry: W_geometry, stroke: "green", strokeWidth: 10,
                    strokeJoin: "bevel", strokeCap: "square" }),
      $(go.Shape, { geometry: W_geometry, stroke: "red", strokeWidth: 10,
                    strokeJoin: "round", strokeCap: "butt" }),
      $(go.Shape, { geometry: W_geometry, stroke: "red", strokeWidth: 10,
                    strokeJoin: "round", strokeCap: "round" }),
      $(go.Shape, { geometry: W_geometry, stroke: "red", strokeWidth: 10,
                    strokeJoin: "round", strokeCap: "square" }),
      $(go.Shape, { geometry: W_geometry, stroke: "purple", strokeWidth: 2,
                    strokeDashArray: [4, 2] }),
      $(go.Shape, { geometry: W_geometry, stroke: "purple", strokeWidth: 2,
                    strokeDashArray: [6, 6, 2, 2] })
    ));

结果：\

[![shape](https://static.yushuangqi.com/blog/2017/0313075900vp2pu4nkaml.jpg)](https://static.yushuangqi.com/blog/2017/0313075900vp2pu4nkaml.jpg)

角度和比例
----------

除了通过`GraphObject.desiredSize`，`GraphObject.width`，`GraphObject.height`这几个属性设置`Shape`的尺寸大小外，还可以使用`GraphObject.angle`和`GraphObject.scale`属性设置`Shape`的角度和比例。

    diagram.add(
    $(go.Part, "Table",
      $(go.Shape, { row: 0, column: 1,
                    figure: "Club", fill: "green", width: 40, height: 40,
                    }),  // default angle is zero; default scale is one
      $(go.Shape, { row: 0, column: 2,
                    figure: "Club", fill: "green", width: 40, height: 40,
                    angle: 30 }),
      $(go.Shape, { row: 0, column: 3,
                    figure: "Club", fill: "green", width: 40, height: 40,
                    scale: 1.5 }),
      $(go.Shape, { row: 0, column: 4,
                    figure: "Club", fill: "green", width: 40, height: 40,
                    angle: 30, scale: 1.5 })
    ));

结果：\

[![shape](https://static.yushuangqi.com/blog/2017/0313075900h2sdvn00k2l.jpg)](https://static.yushuangqi.com/blog/2017/0313075900h2sdvn00k2l.jpg)

下面的例子中，将`Shape.fill`的属性设置为`Brush`画笔对象，并使用了线性渐变画笔给`Shape`填充颜色。

    diagram.add(
    $(go.Part, "Table",
      $(go.Shape, { row: 0, column: 0,
                    figure: "Club", width: 40, height: 40, angle: 0, scale: 1.5,
                    fill: $(go.Brush, go.Brush.Linear, { 0.0: "blue", 1.0: "red" }),
                    background: $(go.Brush, go.Brush.Linear, { 0.0: "yellow", 1.0: "green" }),
                    areaBackground: $(go.Brush, go.Brush.Linear, { 0.0: "gray", 1.0: "lightgray" }) }),
      $(go.Shape, { row: 0, column: 1, width: 10, fill: null, stroke: null }),
      $(go.Shape, { row: 0, column: 2,
                    figure: "Club", width: 40, height: 40, angle: 45, scale: 1.5,
                    fill: $(go.Brush, go.Brush.Linear, { 0.0: "blue", 1.0: "red" }),
                    background: $(go.Brush, go.Brush.Linear, { 0.0: "yellow", 1.0: "green" }),
                    areaBackground: $(go.Brush, go.Brush.Linear, { 0.0: "black", 1.0: "lightgray" }) })
    ));

结果：\

[![shape](https://static.yushuangqi.com/blog/2017/0313075901udvakzdoqmq.jpg)](https://static.yushuangqi.com/blog/2017/0313075901udvakzdoqmq.jpg)

