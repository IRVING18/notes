# 一、Canvas clipX
## 1、canvas.clipRect(left,top,right,bottom);
> 剪切矩形
## 2、canvas.clipPath(path);
> 按path裁剪
- path.setFillType();决定剪裁的是path本身，还是path之外的。   
参考： [path](https://github.com/IRVING18/notes/blob/master/android/自定义View基础/1、Canvas%20Path.md)
## 3、canvas.translate(dx,dy);
> 平移画布
> 注意：在和camera结合使用时，移动camera坐标系需要移动过去画，画完要移动回来不然看不见。不知道为什么，先记下来
## 4、canvas.scale(sx,sy,px,py);
> 缩放   
> sx，sy：xy轴缩放比例   
> px,py: 中心点
## 5、canvas.rotate(degress,px,py)
> 旋转  
> degress: 角度，px，py：中心点
## 6、canvas.skew(sx,sy)
> 错切  
> sx，sy：xy轴错切系数
## 7、canvas.concat(matrix);
> 渲染matrix，矩阵
## 8、canvas和Camera结合使用
> 三维变化  
# 二、Matrix 矩阵
## 1、matrix.postTranslate(dx,dy);
> 平移
例子：
```java
  Matrix matrix = new Matrix();
  canvas.save();
  matrix.postTranslate(100,100);
  canvas.concat(matrix);
  canvas.restore();
```
## 2、matrix.postScale(sx,sy,px,py);
> 缩放
## 3、matrix.postRotate(degress,px,py);
> 旋转  
## 4、matrix.postSkew(sx,sy)
> 错切   
**注意：  
1、matrix.preXX();矩阵前乘   
2、matrix.postXX();矩阵后乘   
3、matrix.setXX();   
有三种，矩阵不太懂 todo 有时间好好学吧**
# 三、Camera 三维变化
## 1、camera.rotateZ(degress);
> 围着Z轴转
**例子：**     
```java
Camera camera = new Camera();
canvas.save();
camera.save();
camera.rotateZ(30);
//投影到canvas
camera.applyToCanvas(canvas);
camera.restore();
canvas.drawBitmap(bitmap,x,y,paint);
canvas.restore();
```



