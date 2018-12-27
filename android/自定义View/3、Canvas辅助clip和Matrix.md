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
## 1、camera.rotateZ(degress);rotateX();rotateY();
> 围着Z轴转
## 注意：rotateX的过程中坐标轴是在转的。
**在画翻页的过程中容易理解些，翻页效果就是围着Y轴转**    
**0-90度画左边，90-180画右边。**
- 1.在rotate之前去clip剪切，需要0-90切左边，90-180切右边。
- 2.在rotate之后去clip剪切，只需要一直都切左边，因为rotate的过程中Y轴一直在转。

![linear](https://github.com/IRVING18/notes/blob/master/android/file/flip.gif)

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
## 2、修正版
> 即从bitmap中间变化，显示成要的效果。

### 2.1 Matrix修正
```java
  centerX：bitmap的中心点x
  centerY: bitmap的中心点y
  canvas.save();
  camera.save();
  //获取matrix矩阵
  camera.getMatrix(matrix);
  camera.restore();
  //矩阵修正
  matrix.preTranslate(-centerX,-centerY);
  matrix.postTranslate(centerX,centerY);
  canvas.concat(matrix);
  canvas.drawBitmap(bitmap,x,y,paint);
  canvas.restore();
```
### 2.2 canvas.translate()修正
```java
  canvas.save();
  camera.save();
  camera.rotateY(30);
  //移动到bitmap中心
  canvas.translate(centerX,centerY);
  //camera投影到canvas
  camera.applyToCanvas(canvas);
  //平移回来
  canvas.translate(-centerX,-centerY);
  camera.restore();
  canvas.drawBitmap(bitmap, point2.x, point2.y, paint);
  canvas.restore();
```
**注
  理解不一定正确**
