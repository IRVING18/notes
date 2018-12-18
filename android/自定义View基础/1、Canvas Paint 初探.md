## 一. Canvas画布
1. canvas.drawColor();
> 画颜色
2. canvas.drawCircle(cx,xy,radius,paint);
> 画圆
> cx，cy，圆心坐标  
> radius 半径
3. canvas.drawRect(left,top,right,bottom,paint);
> 画矩形
> 左上右下角标
4. canvas.drawPoint(x,y,paint);
> 画点
> x,y坐标  
> paint设置strokeWidth（100）
5. canvas.drawOval(left,top,right,bottom,paint);
> 画椭圆
6. canvas.drawLine(startX,startY,stopX,stopY,paint);
> 画线
7. canvas.drawRoundRect(left,top,right,bottom,rx,ry,paint)
> 画圆角矩形
> rx,ry 圆角的半径
8. canvas.drawArc(left,top,right,bottom,startAngle,sweepAngle,useCenter,paint);
> 画弧形，扇形
> startAngle 开始角度
> sweepAngle 滑过的角度
> useCenter 是否填充到圆心  
例：  
![linear](https://github.com/IRVING18/notes/blob/master/android/file/arc.jpg)
9. canvas.drawPath(path,paint);
> 根据path画
## 二、Path 路径
### 第一类
1. addXxx -- 添加子图形  
addCircle(float x, float y, float radius, Direction dir) 添加圆
x, y, radius 这三个参数是圆的基本信息，最后一个参数 dir 是画圆的路径的方向。
> 路径方向有两种：顺时针 (CW clockwise) 和逆时针 (CCW counter-clockwise) 。对于普通情况，这个参数填 CW 还是填 CCW 没有影响。它只是在**需要填充图形** (Paint.Style 为 FILL 或  FILL_AND_STROKE) ，并且**图形出现自相交**时，用于判断填充范围的。比如下面这个图形：
> ![linear](https://github.com/IRVING18/notes/blob/master/android/file/addcircle.png)
