# 三维世界

前几章我们了解了WebGL的工作原理，着色器的使用，矩阵的变换，动画纹理等，

这一章我们主要研究几个问题：

1.以用户视角而进入三维世界

2.控制三维可视空间

3.裁剪

4.处理物体前后关系

5.绘制三角形

## WebGL没有摄像头

WebGL API里是没有摄像头的API的，我们使用一个4*4的矩阵来代替它。

每次我们移动视角，实际上相当于更新了“摄像头”的位置。为此，我们需要相应地对每个vertex做一次变换。同样的，我们也必须保证在每次摄像头移动后法线和光线保持既有的状态。

总而言之，我们需要考虑两种不同的变换：vertex和normal。

## 视线和视点

三维物体具有深度，也就是Z轴，你需要考虑两点: **观察方向**和**可视距离**

我们将观察者所处的地址称为视点，而从视点出发沿着观察方向的射线称作视线

## View transform

view transform将世界原点转换为了视图原点。它根据你的眼睛或是摄像头的位置定位。进行这个变换的矩阵被称为视图矩阵（view matrix）。

![View transform](https://camo.githubusercontent.com/ac4e1a3140c0d48fcc17b477584625e6951fd087/687474703a2f2f67746d7330342e616c6963646e2e636f6d2f7470732f69342f5431654c3151467642615858634e45735f6c2d3738302d3539342e706e67)

为了确定View transform，需要获取三项信息:

1.**视点**是观察者所在的三维空间中位置，视线的起点(eyeX, eyeY, eyeZ)

2.**观察目标点**被观察目标所在的点。视线从视点出发，穿过观察目标点并继续延伸。只有同时知道观察目标点和视点，才能算出视线的方向。观察目标点坐标(atX, atY, atZ)

3.**上方向**是绘制在屏幕上的影像中向上的方向，(upX, upY, upZ)

在WebGL中，我们可以用上述三个矢量创建一个**视图矩阵**，再将该矩阵传给顶点着色器。


### 加入视点的顶点变换

`Matrix4.setLookAt(eyeX, eyeY, eyeZ, atX, atY, atZ, upX, upY, upZ)`

`<视点看到的坐标> = <视图矩阵> * <模型矩阵> * <原始顶点坐标>`

[demo1](http://127.0.0.1:3000/chapter7/lesson1)
[demo2](http://127.0.0.1:3000/chapter7/lesson2)
[demo3](http://127.0.0.1:3000/chapter7/lesson3)

## 可视范围(Projection transform)

当视点在某些位置时，图形会缺失，这是因为我们没有指定**可视范围**，即实际观察得到的区域。

三维物体只有在可视范围内才会被WebGL渲染，不绘制可视范围外的对象是基本的降低开销的手段。

除了水平和垂直外，WebGL还会限制观察者的**可视深度**，这三者共同构成了**可视空间**

### 长方体（盒状）

由正射投影产生，生成的场景可以方便的比较场景中物体大小，适合建筑平面图等技术绘图场景。

![盒状可视空间](../../pic/box_scene.png)

可视空间由前后两个矩形表面确定，分别为**近裁剪面**和**远裁剪面**, canvas上显示的就是可视空间中的物体在**近裁剪面**上的投影。如果裁剪面的宽高比和canvas不一样，那么画面会被压缩并扭曲。近裁剪面和远裁剪面中间的区域就是可视空间，只有在此空间内的物体才会被显示出来。

### 正射投影矩阵(orthogonality matrix)

`Matrix4.setOrtho(left, right, bottom, top, near, far)`

通过各参数计算正射投影矩阵，将其存储在Matrix4中。left和right，top和bottom不一定相等，near与far不能相等

||||
|----|-----|------|
|参数|left,rigght|指定近裁剪面的左边界和右边界|
||bottom,top|指定近裁剪面的上边界和下边界|
||near,far|指定近裁剪面和远裁剪面的位置，即可视空间的近边界和远边界|
|返回值|无||

[demo](http://127.0.0.1:3000/chapter7/lesson4)

### 四棱锥（金字塔）

由透视投影产生，生成的场景看上去更有深度，更自然，更贴近现实，适合大多数真实场景需求。

![透视可视空间](../../pic/perspective_scene.png)

透视投影可视空间也有视点，视线，近裁剪面和远裁剪面

### 透视投影矩阵(perspective matrix)

`Matrix4.setPerspective(fov, aspect, near, far)`

通过各参数计算透视投影矩阵，将其存储在Matrix4中。near与far不能相等，near必须小于far

||||
|----|-----|------|
|参数|fov|制定垂直视角，即可视空间顶面和地面的夹角，必须大于0|
||aspect|制定近裁剪面的宽高比(宽度/高度)|
||near,far|指定近裁剪面和远裁剪面的位置，即可视空间的近边界和远边界|
|返回值|无||

[demo1](http://127.0.0.1:3000/chapter7/lesson5)
[demo2](http://127.0.0.1:3000/chapter7/lesson6)

### 加入可视范围的顶点变换

`<视点看到的坐标> = <投影矩阵> * <视图矩阵> * <模型矩阵> * <原始顶点坐标>`

## 隐藏面消除

在默认情况下，WebGL为了加速绘图操作，是按照顶点在缓冲区的顺序来处理的，后绘制的图形会覆盖已经绘制好的图形，因为这样很搞笑。

隐藏面消除的前提是是正确设置可视空间，否则会产生错误

[demo](http://127.0.0.1:3000/chapter7/lesson7)

### 开启隐藏面消除消除功能

![gl.enable()](../../pic/enable.png)

### 关闭隐藏面消除消除功能

![gl.disable()](../../pic/disable.png)

### 绘制前清除深度缓冲区

在绘制任何一帧之前，都必须清除深度缓冲区，使用`gl.clear(gl.DEPTH_BUFFER_BIT)`，如果需要多个缓冲区，用按位或符号|分割。

## 深度冲突

当几何图形或物体的两个表面极为接近时，有趣深度缓冲区有限的进度不能区分前后，导致表面出现斑驳现象。WebGL提供了多边形偏移来处理这样的问题

[demo](http://127.0.0.1:3000/chapter7/lesson8)

### 开启多边形偏移

`gl.enable(gl.POLYGON_OFFSET_FILL)`

### 绘制之前制定计算偏移量参数

`gl.polygonOffset(1.0, 1.0)`

![gl.polygonOffset()](../../pic/polygonOffset.png)

## 使用顶点索引

### Vertices and Indices

![gl.drawElements()](../../pic/drawElements.png)

Vertices是定义3D对象边界的点。每个vertex都由三个浮点数组成，分别对应X轴，Y轴和Z轴。和OpenGL不同的是，WebGL并没有提供单独操作单个点进行渲染的API，因此我们需要将所有的vertices写在一个JavaScript数组中，并使用它创建WebGL vertex缓存。

Indices是vertices的数字标签（numeric labels）。Indices告诉WebGL如何连接vertices。与vertices相同的是，indices同样也使用数组来创建WebGL index缓存。

> 在WebGL中，有两种WebGL缓存被用作描述和处理对象。包含vertex数据的缓存被称为Vertex Buffer Objects(VBOs)。包含index数据的缓存被称为Index Buffer Objects(IBOs)。 

[demo1](http://127.0.0.1:3000/chapter7/lesson9)
[demo2](http://127.0.0.1:3000/chapter7/lesson10)
