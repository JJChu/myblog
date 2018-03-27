---
title: '[H5] - canvas API 总览'
date: 2018-03-22 16:22:44
tags: [canvas, 可视化]
categories: 你不知道的 HTML
description: canvas api 总览
---
<!-- more -->


## 什么是 canvas

- HTML5 `<canvas>` 元素用于图形的绘制，通过脚本 (通常是JavaScript)来完成；
- 你可以通过多种方法使用 canvas 绘制路径、矩形、圆、字符以及添加图像

## 使用 canvas

### 创建画布

```
// 第一步：创建 canvas 元素
<canvas id="myCanvas" width="200" height="100">浏览器不支持</canvas>

// 第二步：创建绘图上下文 ctx
var canvas = document.getElementById('myCanvas');
if (canvas.getContext) {
    console.log('你的浏览器支持Canvas!');
    var ctx = canvas.getContext('2d');  // 绘制 3D 使用 'webgl'
} else {
    console.log('你的浏览器不支持Canvas!');
}

// 第三步：然后我们可以使用 ctx 对象上的属性和方法进行绘图
```

### 上下文的属性和方法

canvas 是一个二维网格。左上角坐标为 (0,0)

![](https://ws1.sinaimg.cn/large/b6d81e13ly1fpj9mw0brsj2072040glf.jpg)

#### 填充和描边

填充和描边是两个基本操作。填充，就是用指定的样式（颜色、渐变、图像）填充图形；描边，就是只在图形的边缘描线。这两个属性的值可以是字符串，渐变对象或模式对象，默认值为`#000`。

- `ctx.fillStyle = ""` 填充
- `ctx.strokeStyle = ""` 描边
- `ctx.lineWidth = 2` 描边线条的宽度
- `ctx.lineCap = ""` 描边线条末端的形状，平头（butt）、圆头（round）、方头（square）
- `ctx.lineJoin = ""` 控制线条交接的方式，圆交（round）、斜交（bevel）、斜接（miter）

#### 阴影

2D 上下文可以用以下几个值，为形状和路径绘制出阴影：

- `ctx.shadowColor = ""` 用css颜色格式表示的阴影颜色，默认黑色。
- `ctx.shadowOffsetX = ""` 形状或路径 x 轴方向的阴影偏移量，默认 0
- `ctx.shadowOffsetY = ""` 形状或路径 y 轴方向的阴影偏移量，默认 0
- `ctx.shadowBlur = ""` 模糊的像素数，默认 0

#### 渐变

渐变由`CanvasGradient`实例表示。
- `ctx.createLinearGradient(startX,startY,endX,endY)` 创建一个线性渐变
- `ctx.createRadialGradient(startX,startY,radius1,endX,endY,radius2)` 创建一个径向渐变。前三个参数是起点圆的圆心和半径；后三个参数是终点圆的圆心和半径
- `gradient.addColorStop(pos,color)` 添加色标。色标位置`pos`，色标位置是一个 0-1 之间的数字。css颜色值`color`

```
// 1. 首先，我们要创建一个线性渐变。
var gradient = context.createLinearGradient(30, 30, 70, 70);

// 2. 添加色标
gradient.addColorStop(0, "white");
gradient.addColorStop(1, "black");

// 3. 最后我们可以把 fillStyle 或 strokeStyle 设置为这个对象
context.fillStyle = gradient;

// 4. 最后我们就可以用渐变绘制图形了
context.fillRect(30, 30, 50, 50);
```
>值得注意的是，如果要让渐变覆盖到整个矩形，矩形和渐变对象的
坐标必须匹配。否则可能只显示部分效果。因此我们可以用矩形的宽高去反推渐变坐标

#### 模式

模式其实就是重复的图像，可以用来填充或者描边图形。与渐变类似，我们需要用`createPattern()`方法去创建一个新模式

- `ctx.createPattern(img, pattern)` 创建一个新模式
    + 可以使一个 `<img>` 元素、`<video>`元素或者`<canvas>`元素
    + 和一个表示如何重复图像的字符串，与`background-repeat`属性值相同：`repeat` `repeat-x` `repeat-y` `no-repeat`

```
var image = document.images[0],
pattern = context.createPattern(image, "repeat");

context.fillStyle = pattern;
context.fillRect(10, 10, 150, 150);
```
>模式和渐变一样，都是从画布的原点开始的。然后将填充样式（fillStyle）设置为模式对象，只表示在某个特定的区域内显示重复的图像，而不是从某个位置开始绘制重复的图像

#### 绘制路径

**要绘制路径时，** 首先必须调用`ctx.beginPath()`方法，表示要开始绘制新路径。然后再调用下面方法。

- `ctx.moveTo(x,y)` 将绘图游标移动到`(x,y)`，不划线
- `ctx.lineTo(x,y)` 从上一点开始绘制一条直线，到`(x,y)`为止
- `ctx.arc(x,y,radius,startAngle,endAngle,counterclockwise)` 以`(x,y)`为圆心绘制一条弧线，弧线半径为`radius`，起始和结束角度（弧度）为`startAngle`和`endAngle`，最后一个参数表示是否按逆时针方向计算，值为 false。
- `ctx.arcTo(x1,y1,x2,y2,radius)` 从上一点开始绘制一条弧线，到`(x2,y2)`为止，并且以给定的半径`radius`穿过`(x1,y1)`
- `ctx.bezierCurveTo(c1x,c1y,c2x,c2y,x,y)` 从上一点开始绘制一条曲线（贝塞尔曲线），到`(x,y)`为止，并且以`(c1x,c1y)`,`c2x,c2y`为控制点
- `ctx.quadraticCurveTo(cx,cy,x,y)` 从上一点开始绘制一条二次曲线，到`(x,y)`为止，并且以`(cx,cy)`作为控制点
- `ctx.rect(x, y, width, height)` 从点`(x,y)`开始绘制一个矩形。这个方法绘制的是矩形路径，而不是`strokeRect()`和`fillRect()`所绘制的独立的形状
- `ctx.isPointInPath(x,y)` 用于在路径被关闭前确定画布上的某一点`(x,y)`是否位于路径上

**创建路径后，** 接下来可能有以下几种选择：
1. 如果想绘制一条连接到起点的线条（闭环），可以调用`ctx.closePath()`
2. 如果路径已经完成，你想用`fillStyle`填充它，可以调用`ctx.fill()`；当然你想用`strokeStyle`进行描边，可以调用`ctx.stroke()`
3. 还可以调用`ctx.clip()`，可以在路径上创建一个剪切区域

#### 绘制文本

绘制文本的方法：
- `ctx.fillText(text,x,y,width)` 使用`fillStyle`属性绘制文本
- `ctx.strokeText(text,x,y,width)` 使用`strokeStyle`属性绘制文本
- `ctx.measureText(text)` 该方法返回一个`TextMetrics`对象，用以计算文本大小。目前只有`width`属性。

>其中`fillText`和`strokeText`的第四个参数代表文本的最大像素宽度，提供这个参数后，如果传的字符的宽度超出最大像素宽，则绘制的字体在高度上正确，但宽度会收缩至最大像素宽以内。

绘制文本的基础属性：
- `ctx.font = ""` 表示文本样式、大小和字体，与`css`中`font`一致
- `ctx.textAlign = ""` 表示文本对齐方法。值有`start`,`end`,`center`,`left`,`right`
- `ctx.textBaseline = ""` 表示文本的基线。可能的值有`top`,`hanging`,`middle`,`alphabetic`,`ideographic`,`bottom`


#### 绘制矩形

矩形是唯一一种可以直接在 2D 上下文中绘制的形状。

这几个方法都能接受四个参数：x 左边，y 坐标，矩形 width，矩形 height。单位都是像素。

- `ctx.fillRect()` 绘制的矩形会填充指定颜色，颜色通过`fillStyle`属性指定
- `ctx.strokeRect()` 绘制的矩形会使用指定的颜色描边，颜色通过`strokeStyle`属性指定
- `ctx.clearRect()` 用于清除画布上的矩形区域，即变透明

#### 变换

- `ctx.rotate(angle)` 围绕原点旋转图像`angle`弧度
- `ctx.scale(scaleX,scaleY)` 缩放图像，在 x 方向乘以`scaleX`，在 y 方向上乘以`scaleY`，默认都是 1
- `ctx.translate(x,y)` 将坐标原点移动到`(x,y)`。执行这个变换后，坐标`(0,0)`会变成由之前`(x,y)`表示的点。
- `ctx.transform(m1_1, m1_2, m2_1, m2_2, dx, dy)` 直接修改变换矩阵，方式是如下矩阵
- `ctx.setTransform(m1_1, m1_2, m2_1, m2_2, dx, dy)` 将变换矩阵重置为默认状态，然后再调用`transform()`

![](https://ws1.sinaimg.cn/large/b6d81e13ly1fpjlfgjun6j204v01dt8i.jpg)

对于上面的变换，以及`fillStyle` `strokeStyle` 等属性，都会在当前上下文中一直有效，除非修改。虽然没有方法把上下文一切都重置回默认值，但有另个方法可以追踪变化。

- `ctx.save()` 会把当前的所有设置保存进入一个栈结构。需要注意的是，保存的是对绘图上下文的设置和变换，不保存上下文内容
- `ctx.restore()` 在保存设置的栈结构中向前返回一级，回复之前的状态

#### 绘制图像

2D 绘图上下文内置了对图像的支持。我们可以使用`drawImage()`方法在画布上绘制图像、画布或视频。一共可以传入 9 个参数，使用这个方法，有三种不同的参数组合方式。

- `ctx.drawImage(img,oX,oY)` 在画布上绘制图像并定位（原始宽高）
- `ctx.drawImage(img,oX,oY,oW,oH)` 在画布上绘制图像并定位，并且设置图像的宽高
- `ctx.drawImage(img,oX,oY,oW,oH,tX,tY,tW,tH)` 裁剪图像后定位至目标位置并设置宽高。
    + 要绘制的图像`img`，
    + 开始剪切的 x 坐标位置`oX`，开始剪切的 y 坐标位置`oY`，
    + 被剪切图像的宽度`oW`，被剪切图像的高度`oH`，
    + 在画布上放置图像的 x 坐标位置`tX`，在画布上放置图像的 y 坐标位置`tY`，
    + 要使用的图像的宽度`tW`，要使用的图像的高度`tH`

#### 导出图像

- `canvas.toDataURL('image/png')` 导出图像，接受一个参数，即图像的 MIME 类型。这个方法是`canvas`元素上的方法。

```
var drawing = document.getElementById("drawing");

if (drawing.getContext){
    // 取得图像的数据 URI
    var imgURI = drawing.toDataURL("image/png");
    // 显示图像
    var image = document.createElement("img");
    image.src = imgURI;
    document.body.appendChild(image);
}
```

#### 使用图像数据

2D 上下文有一个很好的用法，就是可以通过`getImageData()`取得原始图像数据。

- `ctx.createImageData()` 创建一个`ImageData`对象
    - `createImageData(width,height)` 创建了一个新的具体特定尺寸的ImageData对象。所有像素被预设为透明黑
    - `createImageData(anotherImageData)` 创建一个被`anotherImageData`对象指定的**相同像素**的`ImageData`对象，这个新的对象像素全部被预设为透明黑（并非复制）
- `ctx.getImageData(x,y,width,height)` 该方法返回的对象是一个`ImageData`的实例。
    - 每个对象都有三个属性：`width` `height` `data`。其中`data`是一个数组，保存着每一个像素的数据。
    - 在`data`数组中，一个像素用四个元素保存，分别表示红`data[0]`、绿`data[1]`、蓝`data[2]`、透明值`data[3]`
- `ctx.putImageData(myImageData, dx, dy)` 在场景中写入像素数据

能够直接访问到原始图形数据，就能以各种方式来操作这些数据。比如，像下面这样创建一个简单的灰阶过滤器。

```
var drawing = document.getElementById("drawing");
if (drawing.getContext){
    var context = drawing.getContext("2d"),
        image = document.images[0],
        imageData, data,
        i, len, average,
        red, green, blue, alpha;
    
    context.drawImage(image, 0, 0);
    imageData = context.getImageData(0, 0, image.width,image.height);
    data = imageData.data;
    for (i=0, len=data.length; i < len; i+=4){
        red = data[i];
        green = data[i+1];
        blue = data[i+2];
        alpha = data[i+3];

        average = Math.floor((red + green + blue) / 3);
        data[i] = average; 
        data[i+1] = average; 
        data[i+2] = average;
    }
    imageData.data = data; 
    context.putImageData(imageData, 0, 0);
}
```

#### 合成

2D 上下文中还有两个会应用到所有绘制操作的属性：`globalAlpha` 和 `globalCompositionOperation` 

- `ctx.globalAlpha = ""` 用于指定所有绘制的透明度（0-1），默认 0
    + 如果后续所有操作都要基于相同的透明度，就可以把`globalAlpha`设置为合适值，然后绘制。最后再把它设置回默认 0 
- `ctx.globalCompositionOperation = ""` 表示后绘制的图形怎么与先绘制的图形结合。这个属性的值是字符串
    + `source-over`：默认值。后绘制的图形位于先绘制图形上方。
    + `source-in`：后绘制的图形与先绘制的图形重叠部分可见，其他地方完全透明
    + `source-out`：后绘制的图形与先绘制的图形不重叠的部分可见，先绘制的图形完全透明。
    + `source-atop`：后绘制的图形与先绘制的图形重叠部分可见，先绘制的图形不受影响
    + `destination-over`：后绘制的图形位于先绘制的图形下方
    + `destination-in`：后绘制的图形位于先绘制的图形下方，不重叠的地方完全透明
    + `destination-out`：后绘制的图形擦除与先绘制的的图形重叠的部分
    + `destination-atop`：后绘制的图形位于先绘制的图形下方，在两者不重叠的地方，先绘制的图形会变透明。
    + `lighter`：后绘制的图形与先绘制的图形重叠部分的值相加，使该部分变亮
    + `copy`：后绘制的图形完全替代与之重叠的先绘制的图形
    + `xor`：后绘制的图形与先绘制的图形重叠的部分执行“异或”操作

### 3. WebGL

对于 webgl 暂时不做讨论

