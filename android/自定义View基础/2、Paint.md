# Paint 
# 一、颜色
![linear](https://github.com/IRVING18/notes/blob/master/android/file/color1.jpg)   
## 1、基本颜色
### 1.1 直接设置颜色
- **paint.setColor(int color)**
- **paint.setARGB(int a, int r, int g, int b)**
### 1.2 设置着色器-Shader
- **paint.setShader()**
> LinearGradient 线性渐变
> RadialGradient 辐射渐变
> SweepGradient 扫描渐变
> BitmapShader
> ComposeShader 混合着色器
### 1.3 颜色过滤 setColorFilter
- **paint.setColorFilter()**
#### 1.3.1 LightingColorFilter
LightingColorFilter 的构造方法是 LightingColorFilter(int mul, int add) ，参数里的 mul 和 add 都是和颜色值格式相同的 int 值，其中 mul 用来和目标像素相乘，add 用来和目标像素相加：

```java
R' = R * mul.R / 0xff + add.R  
G' = G * mul.G / 0xff + add.G  
B' = B * mul.B / 0xff + add.B 
```
一个「保持原样」的「基本 LightingColorFilter 」，mul 为 0xffffff，add 为 0x000000（也就是0），那么对于一个像素，它的计算过程就是：
```java
R' = R * 0xff / 0xff + 0x0 = R // R' = R  
G' = G * 0xff / 0xff + 0x0 = G // G' = G  
B' = B * 0xff / 0xff + 0x0 = B // B' = B  
```
基于这个「基本 LightingColorFilter 」，你就可以修改一下做出其他的 filter。比如，如果你想去掉原像素中的红色，可以把它的 mul 改为 0x00ffff （红色部分为 0 ） ，那么它的计算过程就是：
```java
R' = R * 0x0 / 0xff + 0x0 = 0 // 红色被移除  
G' = G * 0xff / 0xff + 0x0 = G  
B' = B * 0xff / 0xff + 0x0 = B  
```
```java
ColorFilter lightingColorFilter = new LightingColorFilter(0x00ffff, 0x000000);  
paint.setColorFilter(lightingColorFilter);  
```

#### 1.3.2 PorterDuffColorFilter
#### 1.3.3 ColorMatrixColorFilter

### 1.4 setXfermode 
- **paint.setXfermode(xfermode);**
> "Xfermode" 其实就是 "Transfer mode"，用 "X" 来代替 "Trans" 是一些美国人喜欢用的简写方式。严谨地讲， Xfermode 指的是你要绘制的内容和 Canvas 的目标位置的内容应该怎样结合计算出最终的颜色。但通俗地说，其实就是要你以绘制的内容作为源图像，以 View 中已有的内容作为目标图像，选取一个 PorterDuff.Mode 作为绘制内容的颜色处理方案。

# 二、效果
## 1. paint.setAntiAlias (boolean aa) 设置抗锯齿
## 2. paint.setStyle(Paint.Style style)
> // FILL 模式，填充  
> // STROKE 模式，画线  
> // FILL_AND_STROKE 模式，填充 + 画线  
## 3.线条形状 
### 3.1 paint.setStrokeWidth(1);  
### 3.2 paintsetStrokeCap(Paint.Cap cap)
> 设置线头的形状。线头形状有三种：BUTT 平头、ROUND 圆头、SQUARE 方头。默认为 BUTT。
![linear](https://github.com/IRVING18/notes/blob/master/android/file/color2.jpg)
### 3.3 .paint.setStrokeJoin(Paint.Join join)
> 设置拐角的形状。有三个值可以选择：MITER 尖角、 BEVEL 平角和 ROUND 圆角。默认为 MITER。
![linear](https://github.com/IRVING18/notes/blob/master/android/file/color3.jpg)
### 3.4 paint.setStrokeMiter(float miter)
> 这个方法是对于 setStrokeJoin() 的一个补充，它用于设置 MITER 型拐角的延长线的最大值。所谓「延长线的最大值」，是这么一回事：
![linear](https://github.com/IRVING18/notes/blob/master/android/file/color4.jpg)
![linear](https://github.com/IRVING18/notes/blob/master/android/file/color5.jpg)
## 4.色彩优化
### 4.1 paint.setFilterBitmap(boolean filter)
设置双线性过滤来优化 Bitmap 放大绘制的效果。
## 5. paint.setPathEffect(PathEffect effect)
> 使用 PathEffect 来给图形的轮廓设置效果。对 Canvas 所有的图形绘制有效，也就是 drawLine() drawCircle() drawPath() 这些方法。大概像这样：
- CornerPathEffect 把所有拐角变成圆角。
- DiscretePathEffect 把线条进行随机的偏离，让轮廓变得乱七八糟。乱七八糟的方式和程度由参数决定。
- DashPathEffect 使用虚线来绘制线条。
- PathDashPathEffect 这个方法比 DashPathEffect 多一个前缀 Path ，所以顾名思义，它是使用一个 Path 来绘制「虚线」。具体看图吧：
![linear](https://github.com/IRVING18/notes/blob/master/android/file/color6.jpg)
- SumPathEffect 这是一个组合效果类的 PathEffect 。它的行为特别简单，就是分别按照两种 PathEffect 分别对目标进行绘制。
- ComposePathEffect 这也是一个组合效果类的 PathEffect 。不过它是先对目标 Path 使用一个 PathEffect，然后再对这个改变后的 Path 使用另一个 PathEffect。
