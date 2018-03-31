
---
date: 2017-03-13T08:20:07+08:00
title: "go_js节点字体设置"
description: ""
disqus_identifier: 1489364407182816032
slug: "go_jsjie-dian-zi-ti-she-zhi"
source: "https://my.oschina.net/u/2391658/blog/856451"
tags: 
- gojs 
categories:
- 编程语言与开发
---

`TextBlock`是用于显示文本信息的对象。

通过设置`TexkBlock.text`属性来显示文本信息，这也是唯一的一个方法。因为`TexkBlock`继承自`GraphObject`，所以一些`GraphObject`的属性也有可能对文本有影响。

字体和颜色
----------

可以通过`TexkBlock.font`属性设置文本的字体，该属性的值可以使用CSS来设置。

可以通过`TextBlock.stroke`属性设置文本字体的颜色，同样可以使用CSS来设置。

因为`TexkBlock`继承自`GraphObject`，所以`GraphObject.background`属性也可以作用于`TextBlock`，可以通过该属性设置文本背景色。

    diagram.add(
    $(go.Part, "Vertical",
      $(go.TextBlock, { text: "a Text Block" }),
      $(go.TextBlock, { text: "a Text Block", stroke: "red" }),
      $(go.TextBlock, { text: "a Text Block", background: "lightblue" }),
      $(go.TextBlock, { text: "a Text Block", font: "bold 14pt serif" })
    ));

结果：\

[![textblock](https://static.yushuangqi.com/blog/2017/0313075858ur2lqgsagip.jpg)](https://static.yushuangqi.com/blog/2017/0313075858ur2lqgsagip.jpg)

尺寸和裁剪
----------

`TexkBlock`的自然尺寸是会自适应设置文本的字体以及文本长度的。但是实际上它的尺寸是可大可小的。

下面的例子中首先展示了自然尺寸的`TextBlock`，然后对其进行明确的尺寸设置，并给与绿色背景：

    diagram.add(
    $(go.Part, "Vertical",
      $(go.TextBlock, { text: "a Text Block", background: "lightgreen", margin: 2 }),
      $(go.TextBlock, { text: "a Text Block", background: "lightgreen", margin: 2,
                        width: 100, height: 33 }),
      $(go.TextBlock, { text: "a Text Block", background: "lightgreen", margin: 2,
                        width: 60, height: 33 }),
      $(go.TextBlock, { text: "a Text Block", background: "lightgreen", margin: 2,
                        width: 50, height: 22 }),
      $(go.TextBlock, { text: "a Text Block", background: "lightgreen", margin: 2,
                        width: 40, height: 9 })
    ));

结果：\

[![textblock](https://static.yushuangqi.com/blog/2017/0313075858f2mbxwauqsw.jpg)](https://static.yushuangqi.com/blog/2017/0313075858f2mbxwauqsw.jpg)

文本自适应
----------

`TextBlock`也可以使文本信息在规定的尺寸内自动换行，以达到自适应尺寸。可以通过`TextBlock.wrap`属性来设置，该属性不能为空，必须对其进行属性设置。

下面的例子中，第一个使用自然尺寸，第二个规定了宽度，第三第四个在规定相同宽度的基础上设置了`TextBlock.wrap`属性：

    diagram.add(
    $(go.Part, "Vertical",
      $(go.TextBlock, { text: "a Text Block", background: "lightgreen", margin: 2 }),
      $(go.TextBlock, { text: "a Text Block", background: "lightgreen", margin: 2,
                        width: 50, wrap: go.TextBlock.None }),
      $(go.TextBlock, { text: "a Text Block", background: "lightgreen", margin: 2,
                        width: 50, wrap: go.TextBlock.WrapDesiredSize }),
      $(go.TextBlock, { text: "a Text Block", background: "lightgreen", margin: 2,
                        width: 50, wrap: go.TextBlock.WrapFit })
    ));

结果：\

[![textblock](https://static.yushuangqi.com/blog/2017/0313075859jn3ykj1q353.jpg)](https://static.yushuangqi.com/blog/2017/0313075859jn3ykj1q353.jpg)

文本对齐
--------

使用`TextBlock.textAlign`属性可以设置文本的对齐方式。

这里注意`TextBlock.textAlign`与`GraphObject.alignment`是不同的，前者是针对文本的对齐方式，后者是针对所在容器的对齐方式：

    diagram.add(
    $(go.Part, "Horizontal",
      $(go.Panel, "Vertical",
        { width: 150, defaultStretch: go.GraphObject.Horizontal },
        $(go.TextBlock, { text: "textAlign: 'left'", background: "lightgreen", margin: 2,
                          textAlign: "left" }),
        $(go.TextBlock, { text: "textAlign: 'center'", background: "lightgreen", margin: 2,
                          textAlign: "center" }),
        $(go.TextBlock, { text: "textAlign: 'right'", background: "lightgreen", margin: 2,
                          textAlign: "right" })
      ),
      $(go.Panel, "Vertical",
        { width: 150, defaultStretch: go.GraphObject.None },
        $(go.TextBlock, { text: "alignment: Left", background: "lightgreen", margin: 2,
                          alignment: go.Spot.Left }),
        $(go.TextBlock, { text: "alignment: Center", background: "lightgreen", margin: 2,
                          alignment: go.Spot.Center }),
        $(go.TextBlock, { text: "alignment: Right", background: "lightgreen", margin: 2,
                          alignment: go.Spot.Right })
      )
    ));

结果：\

[![textblock](https://static.yushuangqi.com/blog/2017/0313075859stddgfapcms.jpg)](https://static.yushuangqi.com/blog/2017/0313075859stddgfapcms.jpg)

对齐方式、换行、自适应
----------------------

`TextBlock.textAlign`不管在自然尺寸中处理多行还是在规定尺寸中处理多行都很好用。

`TextBlock.isMultiline`属性用于设置是否开启内嵌文本中的换行符作用。

`TextBlock.wrap`属性在处理换行时就更加游刃有余，它会根据`TexkBlock`的尺寸自动对文本进行换行。

    diagram.add(
    $(go.Part, "Vertical",
      $(go.TextBlock, { text: "a Text Block\nwith three logical lines\nof text",
                        background: "lightgreen", margin: 2,
                        isMultiline: false }),
      $(go.TextBlock, { text: "a Text Block\nwith three logical lines\nof text",
                        background: "lightgreen", margin: 2,
                        isMultiline: true }),
      $(go.TextBlock, { text: "a Text Block\nwith three logical lines\nof centered text",
                        background: "lightgreen", margin: 2,
                        isMultiline: true, textAlign: "center" }),
      $(go.TextBlock, { text: "a single line of centered text that should wrap because we will limit the width",
                        background: "lightgreen", margin: 2, width: 80,
                        wrap: go.TextBlock.WrapFit, textAlign: "center" })
    ));

结果：\

[![textblock](https://static.yushuangqi.com/blog/2017/0313075900glrjdmoxz21.jpg)](https://static.yushuangqi.com/blog/2017/0313075900glrjdmoxz21.jpg)

 

编辑文本
--------

GOJS也提供了对`TextBlock`文本的编辑功能，只需要设置`TextBlock.editable`为`true`即可。\

如果你想对`TextBlock`中的文本进行某种规则的验证，那么可以设置`TextBlock.textValidation`属性，该属性的值为`function`，你可以自行编写验证规则。你甚至可以更换文本编辑器，设置`TextBlock.textEditor`属性即可。

    diagram.add(
    $(go.Part,
      $(go.TextBlock,
        { text: "select and then click to edit",
          background: "lightblue",
          editable: true, isMultiline: false })
    ));
    diagram.add(
    $(go.Part,
      $(go.TextBlock,
        { text: "this one allows embedded newlines",
          background: "lightblue",
          editable: true })
    ));

结果：\

[![textblock](https://static.yushuangqi.com/blog/2017/0313075900df2nevy5qcn.jpg)](https://static.yushuangqi.com/blog/2017/0313075900df2nevy5qcn.jpg)

