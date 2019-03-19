# FlipBoard 翻页动画效果

![linear](https://github.com/IRVING18/notes/blob/master/android/file/filpview.gif)
## 一、动画分析
> 方法：可以解开动图然后观察

- 1. 第一步肯定是抬起右边。
- 2. 让这个状态围着Z轴旋转。但是还不能让图旋转
- 3. 旋转270度之后，抬起上部分。
## 二、实现思路
> 注意：camera是需要矫正的。
#### 一半是翻起的状态
1. 用camera实现抬起右边，围着Y轴转-45度，就可以抬起
2. 最重要的一步，画抬起的一半，首先思路是这样的，就像移动坐标原点一样，这个抬起的状态旋转但是图片不旋转的效果也可以这么实现。
2.1 先让canvas围着Z轴，也就是平面旋转，然后旋转的时候把camera投影到canvas，然后顺便剪裁右边。因为旋转的是坐标轴，所以只剪裁右边就ok。
2.2 这时候不做操作那就是这个图转了，但是神奇的地方来了。这时候可以让canvas围着Z轴旋转回来。然后图片就没有旋转，但是抬起状态却保留下来了。
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


