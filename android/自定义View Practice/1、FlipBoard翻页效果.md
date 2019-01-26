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
#### 10. drawBitmap(Bitmap bitmap, float left, float top, Paint paint) 画 Bitmap
> left,top是图片左上角坐标
#### 11. canvas.save();
> 保存画布位置
#### 12. canvas.restore();
> 还原画布位置，回到save时
#### 13. Canvas.saveLayer() 
> 来设置离屏缓冲  
> 场景： 在paint.setXfermode()的时候要用，不然渲染出来的会有问题
#### 14. canvas
## 二、Path 路径
## Path 方法第一类：直接描述路径
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
![linear](https://github.com/IRVING18/notes/blob/master/android/file/arcToForceMoveFalse.jpg)

forceMoveTo = true    
![linear](https://github.com/IRVING18/notes/blob/master/android/file/arcToForceMoveTrue.jpg)

#### 6. close() 和 lineTo(起点坐标) 是完全等价的。

## Path 方法第二类：辅助的设置或计算
### Path.setFillType(Path.FillType ft) 设置填充方式  
前面在说 dir 参数的时候提到， Path.setFillType(fillType) 是用来设置图形自相交时的填充算法的：  
![linear](https://github.com/IRVING18/notes/blob/master/android/file/arc1.jpg)  
方法中填入不同的 FillType 值，就会有不同的填充效果。FillType 的取值有四个：
- EVEN_ODD  
- WINDING （默认值）  
- INVERSE_EVEN_ODD  
- INVERSE_WINDING  

其中后面的两个带有 INVERSE_ 前缀的，只是前两个的反色版本，所以只要把前两个，即 EVEN_ODD 和  WINDING，搞明白就可以了。  

EVEN_ODD 和 WINDING 的原理有点复杂，直接讲出来的话信息量太大，所以我先给一个简单粗暴版的总结，你感受一下： WINDING 是「全填充」，而 EVEN_ODD 是「交叉填充」：  

![linear](https://github.com/IRVING18/notes/blob/master/android/file/arc2.jpg)  
之所以叫「简单粗暴版」，是因为这些只是通常情形下的效果；而如果要准确了解它们在所有情况下的效果，就得先知道它们的原理，即它们的具体算法。

### EVEN_ODD 和 WINDING 的原理
#### EVEN_ODD
> 即 even-odd rule （奇偶原则）：对于平面中的任意一点，向任意方向射出一条射线，这条射线和图形相交的次数（相交才算，相切不算哦）如果是奇数，则这个点被认为在图形内部，是要被涂色的区域；如果是偶数，则这个点被认为在图形外部，是不被涂色的区域。还以左右相交的双圆为例：

![linear](https://github.com/IRVING18/notes/blob/master/android/file/arc3.jpg)  

> 射线的方向无所谓，同一个点射向任何方向的射线，结果都是一样的，不信你可以试试。
> 从上图可以看出，射线每穿过图形中的一条线，内外状态就发生一次切换，这就是为什么 EVEN_ODD 是一个「交叉填充」的模式。

#### WINDING
> 即 non-zero winding rule （非零环绕数原则）：首先，它需要你图形中的所有线条都是有绘制方向的：  
![linear](https://github.com/IRVING18/notes/blob/master/android/file/arc4.jpg)   

> 然后，同样是从平面中的点向任意方向射出一条射线，但计算规则不一样：以 0 为初始值，对于射线和图形的所有交点，遇到每个顺时针的交点（图形从射线的左边向右穿过）把结果加 1，遇到每个逆时针的交点（图形从射线的右边向左穿过）把结果减 1，最终把所有的交点都算上，得到的结果如果不是 0，则认为这个点在图形内部，是要被涂色的区域；如果是 0，则认为这个点在图形外部，是不被涂色的区域。

![linear](https://github.com/IRVING18/notes/blob/master/android/file/arc6.jpg)  

> 和 EVEN_ODD 相同，射线的方向并不影响结果。

所以，我前面的那个「简单粗暴」的总结，对于 WINDING 来说并不完全正确：如果你所有的图形都用相同的方向来绘制，那么 WINDING 确实是一个「全填充」的规则；但如果使用不同的方向来绘制图形，结果就不一样了。

图形的方向：对于添加子图形类方法（如 Path.addCircle() Path.addRect()）的方向，由方法的  dir 参数来控制，这个在前面已经讲过了；而对于画线类的方法（如 Path.lineTo() Path.arcTo()）就更简单了，线的方向就是图形的方向。

所以，完整版的 EVEN_ODD 和 WINDING 的效果应该是这样的：

![linear](https://github.com/IRVING18/notes/blob/master/android/file/arc5.jpg)  

而 INVERSE_EVEN_ODD 和 INVERSE_WINDING ，只是把这两种效果进行反转而已，你懂了 EVEN_ODD 和  WINDING ，自然也就懂 INVERSE_EVEN_ODD 和 INVERSE_WINDING 了，我就不讲了。

好，花了好长的篇幅来讲 drawPath(path) 和 Path，终于讲完了。同时， Canvas 对图形的绘制就也讲完了。图形简单时，使用 drawCircle() drawRect() 等方法来直接绘制；图形复杂时，使用  drawPath() 来绘制自定义图形。


