# Paint 
# 一、颜色
![linear](https://github.com/IRVING18/notes/blob/master/android/file/color1.jpg)   
## 1、基本颜色
### 1.1 直接设置颜色
- paint.setColor(int color)
- paint.setARGB(int a, int r, int g, int b)
### 1.2 设置着色器-Shader
- LinearGradient 线性渐变
- RadialGradient 辐射渐变
- SweepGradient 扫描渐变
- BitmapShader
- ComposeShader 混合着色器
#### 1.2.1 LinearGradient
> LinearGradient(float x0, float y0, float x1, float y1, int color0, int color1, Shader.TileMode tile)  
> x0 y0 x1 y1：渐变的两个端点的位置   
> color0 color1 是端点的颜色   
> tile：端点范围之外的着色规则，类型是 TileMode。  
> CLAMP, MIRROR 和 REPEAT。CLAMP会在端点之外延续端点处的颜色；MIRROR 是镜像模式；REPEAT 是重复模式。具体的看一下例子就明白。  

```java
Shader shader = new LinearGradient(100, 100, 500, 500, Color.parseColor("#E91E63"),  
        Color.parseColor("#2196F3"), Shader.TileMode.CLAMP);
paint.setShader(shader);
...
canvas.drawCircle(300, 300, 200, paint);  
```
![linear](https://github.com/IRVING18/notes/blob/master/android/file/color2.jpg) 

#### 1.2.2 RadialGradient
> 构造方法：   
RadialGradient(float centerX, float centerY, float radius, int centerColor, int edgeColor, TileMode tileMode)。

参数： 
centerX centerY：辐射中心的坐标   
radius：辐射半径   
centerColor：辐射中心的颜色   
edgeColor：辐射边缘的颜色   
tileMode：辐射范围之外的着色模式。  

```java
Shader shader = new RadialGradient(300, 300, 200, Color.parseColor("#E91E63"),  
        Color.parseColor("#2196F3"), Shader.TileMode.CLAMP);
paint.setShader(shader);
...
canvas.drawCircle(300, 300, 200, paint);  
```
![linear](https://github.com/IRVING18/notes/blob/master/android/file/color3.jpg) 

#### 1.2.3 SweepGradient 
> 构造方法：   
SweepGradient(float cx, float cy, int color0, int color1)  

参数：   
cx cy ：扫描的中心   
color0：扫描的起始颜色   
color1：扫描的终止颜色  

```java
Shader shader = new SweepGradient(300, 300, Color.parseColor("#E91E63"),  
        Color.parseColor("#2196F3"));
paint.setShader(shader);

...

canvas.drawCircle(300, 300, 200, paint);  
```

![linear](https://github.com/IRVING18/notes/blob/master/android/file/color4.jpg) 

#### 1.2.4 BitmapShader
> 用 Bitmap 来着色（终于不是渐变了）。其实也就是用 Bitmap 的像素来作为图形或文字的填充。大概像这样：

> 构造方法：   
BitmapShader(Bitmap bitmap, Shader.TileMode tileX, Shader.TileMode tileY)

参数：   
bitmap：用来做模板的 Bitmap 对象   
tileX：横向的 TileMode   
tileY：纵向的 TileMode。  
 
```java
Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.batman);  
Shader shader = new BitmapShader(bitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);  
paint.setShader(shader);

...

canvas.drawCircle(300, 300, 200, paint);  
```

![linear](https://github.com/IRVING18/notes/blob/master/android/file/color5.jpg) 

#### 1.2.5 ComposeShader 混合着色器
> 构造方法：ComposeShader(Shader shaderA, Shader shaderB, PorterDuff.Mode mode)  

参数：   
shaderA, shaderB：两个相继使用的 Shader   
mode: 两个 Shader 的叠加模式，即 shaderA 和 shaderB 应该怎样共同绘制。它的类型是 PorterDuff.Mode 。  

```java
// 第一个 Shader：头像的 Bitmap
Bitmap bitmap1 = BitmapFactory.decodeResource(getResources(), R.drawable.batman);  
Shader shader1 = new BitmapShader(bitmap1, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);

// 第二个 Shader：从上到下的线性渐变（由透明到黑色）
Bitmap bitmap2 = BitmapFactory.decodeResource(getResources(), R.drawable.batman_logo);  
Shader shader2 = new BitmapShader(bitmap2, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);

// ComposeShader：结合两个 Shader
Shader shader = new ComposeShader(shader1, shader2, PorterDuff.Mode.SRC_OVER);  
paint.setShader(shader);

...

canvas.drawCircle(300, 300, 300, paint);  
```
> 注意：上面这段代码中我使用了两个 BitmapShader 来作为 ComposeShader() 的参数，而 ComposeShader() 在硬件加速下是不支持两个相同类型的 Shader 的，所以这里也需要关闭硬件加速才能看到效果。

![linear](https://github.com/IRVING18/notes/blob/master/android/file/color6.jpg) 
