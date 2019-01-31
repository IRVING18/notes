# 水波纹效果
![linear](https://thumbnail0.baidupcs.com/thumbnail/701cac2ef01428b8747d82d0f8cb8213?fid=1313615635-250528-819341963372592&time=1548918000&rt=sh&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-en8wjKJ4MNcEXWT9LdCo5vJtPaY%3D&expires=8h&chkv=0&chkbd=0&chkpc=&dp-logid=710939205077419547&dp-callid=0&size=c710_u400&quality=100&vuk=-&ft=video)
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


