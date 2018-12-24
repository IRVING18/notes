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
- <1> LightingColorFilter
- <2> PorterDuffColorFilter
- <3> ColorMatrixColorFilter

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
## 6. paint.setShadowLayer(float radius, float dx, float dy, int shadowColor)
> 设置阴影
## 7. paint.setMaskFilter(MaskFilter maskfilter) 
- BlurMaskFilter 模糊效果的 MaskFilter。
- EmbossMaskFilter 浮雕效果的 MaskFilter。 
## 8.paint.getPath() 获取绘制的 Path
- getFillPath(Path src, Path dst) //从src的path中拿出绘制线路放在dst的path中
- getTextPath(String text, int start, int end, float x, float y, Path path) / getTextPath(char[] text, int index, int count, float x, float y, Path path)

# 三、drawText()相关
## 1.paint.setTextSize(60); 
> 设置文字大小
## 2.绘制自动换行文字
> width:宽度  
> align:对齐方式 Layout.Alignment.ALIGN_NORMAL..  
> spacingmult:行距倍数  
> spacingadd:行距距离增加多少  

```java
StaticLayout staticLayout = new StaticLayout(text,textPaint,width,align,spacingmult,spacingadd,true);
staticLayout.draw(canvas);
```
## 3.paint.setTypeface();
> 设置文字样式  
> Typeface.DEFAULT  
> Typeface.SERIF 宋体
> Typeface.createFromAsset(getContext().getAssets(),"Satisfy-Regular.ttf")   
> 在资源目录中获取字体样式
## 4.Paint.setFakeBoldText() 
> 来加粗文字  
## 5.Paint.setStrikeThruText() 
> 来设置删除线  
## 6.Paint.setUnderlineText()
> 来设置下划线
## 7.Paint.setTextSkewX() 
> 来让文字倾斜
## 8.Paint.setTextScaleX()
> 来改变文字宽度
## 9.Paint.setTextAlign() 
> 来调整文字对齐方式   
> Paint.Align.LEFT 左对齐  
> Paint.Align.CENTER 居中  
> Paint.Align.RIGHT  右对齐
## 10.Paint.getFontSpacing() 
> 来获取推荐的行距
## 11.Paint.measureText 
> 测量出文字宽度，让文字可以相邻绘制   
```java
canvas.drawText(text1, 50, 200, paint1);
canvas.drawText(text2, 50 + paint1.measureText(text1), 200, paint2);
canvas.drawText(text3, 50 + paint1.measureText(text1) + paint2.measureText(text2), 200, paint1);
```
## 12.Paint.getTextBounds(String text, int start, int end, Rect bounds)
> 获取文字范围的rect  
> text :文字内容  
> start: 开始测量角标  
> end : 结束角标  
> Rect: 测量完成注入rect返回  
注：  
bounds.top 和 bounds.bottom 的y轴坐标原点是baseline  
应用：  
- 用于文字居中
- 比如期望文字在middle位置居中，

# 四、初始化类
## 1. paint.reset()
重置 Paint 的所有属性为默认值。相当于重新 new 一个，不过性能当然高一些啦。
## 2. paint.set(Paint src)
把 src 的所有属性全部复制过来。相当于调用 src 所有的 get 方法，然后调用这个 Paint 的对应的 set 方法来设置它们。
## 3. paint.setFlags(int flags)
批量设置 flags。相当于依次调用它们的 set 方法。例如： 
```java
paint.setFlags(Paint.ANTI_ALIAS_FLAG | Paint.DITHER_FLAG);  
```

这行代码，和下面这两行是等价的：  

```java
paint.setAntiAlias(true);  
paint.setDither(true);  
```
