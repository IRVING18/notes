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
- 

