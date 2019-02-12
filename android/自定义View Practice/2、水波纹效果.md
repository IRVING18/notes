# 水波纹效果
![linear](https://picabstract-preview-ftn.weiyun.com/ftn_pic_abs_v3/cc46587bcbff42dea0025d4f474de4509bf2cbfa1ffba4fdb6f1bc62797d16b6c117c95d878ec57bc21589e438fb97d4?pictype=scale&amp;from=30113&amp;version=3.3.3.3&amp;uin=1141502381&amp;fname=QQ20190130-164311-HD.gif&amp;size=750)
## 一、动画分析
-  画圈圈，越往外画的圈颜色越淡
## 二、实现思路
> 学习：Interpolator插值器的用法，getInterpolation(percent百分比)能获取插值器相关参数
#### 画圈圈
1. canvas.drawCircle()就可以
2. 怎么控制画自定多个circle画呢，可以用list存起来，然后遍历list 然后draw
3. 怎么控制circle取消，只需要给个边界值，超出边界的circle就remove掉。这里使用的是以时间2s做的边界值。

## 三、实现代码

[具体项目地址](https://github.com/IRVING18/RippleView)

关键代码：

```java
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        centerX = getWidth() / 2;
        centerY = getHeight() / 2;

        maxRadius = getWidth() / 2;

        Iterator<Circle> iterator = circleList.iterator();
        while (iterator.hasNext()) {
            Circle circle        = iterator.next();
            int    alpha         = circle.getAlpha();
            float  currentRidius = circle.getCurrentRidius();
            long   creatTime     = circle.getmCreatTime();
            //如果创建时间小于 mDuration再画，剩下的移除
            if (System.currentTimeMillis() - creatTime < mDuration) {
                paint.setAlpha(alpha);
                canvas.drawCircle(centerX, centerY, currentRidius, paint);
            } else {
                iterator.remove();
            }
        }

        Paint.FontMetrics fontMetrics = textPaint.getFontMetrics();
        textPaint.setTextSize(50);
        canvas.drawText(text, centerX, centerY - (fontMetrics.descent + fontMetrics.ascent) / 2, textPaint);

        if (circleList.size() > 0) {
            postInvalidateDelayed(10);
        }
    }
    
    
    
    public class Circle {
        private long mCreatTime;

        public Circle(long mCreatTime) {
            this.mCreatTime = mCreatTime;
        }

        public long getmCreatTime() {
            return mCreatTime;
        }

        public int getAlpha() {
            //计算透明度，越往外越透明
            float precent = (System.currentTimeMillis() - mCreatTime) * 1.0f / mDuration;
            return (int) (255 - mInterpolator.getInterpolation(precent) * 255);
        }

        public float getCurrentRidius() {
            //计算半径，(System.currentTimeMillis() - mCreatTime) 大于 mDuration就进不来这了
            float precent = (System.currentTimeMillis() - mCreatTime) * 1.0f / mDuration;
            return mixRadius + mInterpolator.getInterpolation(precent) * (maxRadius - mixRadius);
        }
    }
```


