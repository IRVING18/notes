# FlipBoard 翻页动画效果

![linear](https://github.com/rengwuxian/MaterialEditText/blob/master/images/floating_label.png)
## 一、动画分析
- 1. 文字一个简单的渐变alpha 0->1。
- 2. y轴坐标的移动。
- 3. 唯一的关键就是设置padding，来让顶部腾出位置来绘制文字。
## 二、实现思路

#### 设置一个fraction值，用来同时控制alpha和y轴坐标的渐变。
1. alpha
#### 另一半是不动的
3. 第三步画不动的一半。思路和第二步类似了。

## 三、实现代码

[具体项目地址](https://github.com/IRVING18/FlipBoardView)

关键代码：

```java
        //画变化一半
        canvas.save();
        mCamera.save();
        //移动到中心点
        canvas.translate(centerX, centerY);
        //围着Z轴旋转
        canvas.rotate(degreeZ);

        //围着Y轴旋转，抬起右边
        mCamera.rotateY(degreeY);
        //投影到canvas
        mCamera.applyToCanvas(canvas);

        //裁剪右边,此时的坐标轴在centerX,centerY处
        canvas.clipRect(0, -centerY, centerX, centerY);

        //旋转回来，图片转回来但是抬起状态会保留在原来的坐标轴处
        canvas.rotate(-degreeZ);
        //平移回来
        canvas.translate(-centerX, -centerY);

        //画bitmap
        canvas.drawBitmap(bitmap, bpLeft, bpTop, paint);
        mCamera.restore();
        canvas.restore();


        //画不动的一半
        canvas.save();
        mCamera.save();
        canvas.translate(centerX, centerY);
        //和变化的一半一样也是旋转，然后剪切另一半
        canvas.rotate(degreeZ);
        //剪切不变化的一半,此时坐标轴原点在centerX,centerY
        canvas.clipRect(-centerX, -centerY, 0, centerY);

        //最终抬起动作
        //因为设置动画的时候已经是degreeZ为270,才会执行第三个抬起动作动画，所以这是的坐标系已经变化了，横着的就是Y轴
        mCamera.rotateY(degreeLast);
        mCamera.applyToCanvas(canvas);

        canvas.rotate(-degreeZ);
        canvas.translate(-centerX, -centerY);

        canvas.drawBitmap(bitmap, bpLeft, bpTop, paint);
        mCamera.restore();
        canvas.restore();
```


