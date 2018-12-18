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
