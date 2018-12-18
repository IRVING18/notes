## 一. Canvas画布
#### 1. canvas.drawColor();
> 画颜色
#### 2. canvas.drawCircle(cx,xy,radius,paint);
> 画圆
> cx，cy，圆心坐标  
> radius 半径
#### 3. canvas.drawRect(left,top,right,bottom,paint);
> 画矩形
> 左上右下角标
#### 4. canvas.drawPoint(x,y,paint);
> 画点
> x,y坐标  
> paint设置strokeWidth（100）
#### 5. canvas.drawOval(left,top,right,bottom,paint);
> 画椭圆
#### 6. canvas.drawLine(startX,startY,stopX,stopY,paint);
> 画线
#### 7. canvas.drawRoundRect(left,top,right,bottom,rx,ry,paint)
> 画圆角矩形
> rx,ry 圆角的半径
#### 8. canvas.drawArc(left,top,right,bottom,startAngle,sweepAngle,useCenter,paint);
> 画弧形，扇形
> startAngle 开始角度
> sweepAngle 滑过的角度
> useCenter 是否填充到圆心  
例：  
![linear](https://github.com/IRVING18/notes/blob/master/android/file/arc.jpg)
#### 9. canvas.drawPath(path,paint);
> 根据path画
## 二、Path 路径
### 第一类 addXxx -- 添加子图形  
1. addCircle(float x, float y, float radius, Direction dir) 添加圆
x, y, radius 这三个参数是圆的基本信息，最后一个参数 dir 是画圆的路径的方向。
> 路径方向有两种：顺时针 (CW clockwise) 和逆时针 (CCW counter-clockwise) 。对于普通情况，这个参数填 CW 还是填 CCW 没有影响。它只是在**需要填充图形** (Paint.Style 为 FILL 或  FILL_AND_STROKE) ，并且**图形出现自相交**时，用于判断填充范围的。比如下面这个图形：  
> ![linear](https://github.com/IRVING18/notes/blob/master/android/file/addcircle.png)
### 第二类 xxxTo()
> 这一组和第一组 addXxx() 方法的区别在于，第一组是添加的完整封闭图形（除了 addPath() ），而这一组添加的只是一条线。
#### 1. lineTo(float x, float y) / rLineTo(float x, float y) 画直线
> lineTo: 相对起点默认是view的0，0  
> rLineTo: 相对当前位置
#### 2. quadTo(float x1, float y1, float x2, float y2) / rQuadTo(float dx1, float dy1, float dx2, float dy2) 画二次贝塞尔曲线  

这条二次贝塞尔曲线的起点就是当前位置，而参数中的 x1, y1 和 x2, y2 则分别是控制点和终点的坐标。和 rLineTo(x, y) 同理，rQuadTo(dx1, dy1, dx2, dy2) 的参数也是相对坐标
> 贝塞尔曲线：贝塞尔曲线是几何上的一种曲线。它通过起点、控制点和终点来描述一条曲线，主要用于计算机图形学。概念总是说着容易听着难，总之使用它可以绘制很多圆润又好看的图形，但要把它熟练掌握、灵活使用却是不容易的。不过还好的是，一般情况下，贝塞尔曲线并没有什么用处，只在少数场景下绘制一些特殊图形的时候才会用到，所以如果你还没掌握自定义绘制，可以先把贝塞尔曲线放一放，稍后再学也完全没问题。至于怎么学，贝塞尔曲线的知识网上一搜一大把，我这里就不讲了。 

#### 3. cubicTo(float x1, float y1, float x2, float y2, float x3, float y3) / rCubicTo(float x1, float y1, float x2, float y2, float x3, float y3) 画三次贝塞尔曲线
#### 4. moveTo(float x, float y) / rMoveTo(float x, float y) 移动到目标位置  

不论是直线还是贝塞尔曲线，都是以当前位置作为起点，而不能指定起点。但你可以通过 moveTo(x, y) 或 rMoveTo() 来改变当前位置，从而间接地设置这些方法的起点。

#### 5.  arcTo(RectF oval, float startAngle, float sweepAngle, boolean forceMoveTo) / arcTo(float left, float top, float right, float bottom, float startAngle, float sweepAngle, boolean forceMoveTo) / arcTo(RectF oval, float startAngle, float sweepAngle) 

> 画弧形
> arcTo() 和 addArc()。它们也是用来画线的，但并不使用当前位置作为弧线的起点。
> forceMoveTo 绘制是要「抬一下笔移动过去」，还是「直接拖着笔过去」

forceMoveTo = false  
![linear](https://github.com/IRVING18/notes/blob/master/android/file/arcToForceMoveFalse.png)

forceMoveTo = true    
![linear](https://github.com/IRVING18/notes/blob/master/android/file/arcToForceMoveTrue.png)
