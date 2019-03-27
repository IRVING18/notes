# ScaleIamgeView 手势缩放图片

# 一、实现思路
#### 1、实现双击缩放 
- 1.1 canvas.**scale()** 来实现 
- 1.2 监听双击事件 GestureDetectorCompat 的 **OnDoubleTapListener** 的 **onDoubleTapEvent()**方法
- 1.3 同时通过控制scaleFraction系数，来控制一个动画，让变化过程更柔顺。
#### 2、实现拖动
- 2.1 canvas.**translate()** 来实现
- 2.2 监听触摸事件 GestureDetectorCompat 的 **OnGestureListener** 的 **onScroll()** 方法可以监听手势滑动。
#### 3、实现惯性滑动
- 3.1 监听触摸事件 GestureDetectorCompat 的 **OnGestureListener** 的 **onFling()** 方法可以监听手势惯性。
- 3.2 通过 **OverScroller** 惯性计算模型，可以帮助计算惯性值。

[基础知识](https://github.com/IRVING18/notes/blob/master/android/自定义View/A12、手势触摸-初探.md)

# 二、实现代码

[具体项目](https://github.com/IRVING18/PhotoViewSimple)

关键代码  
## 1、实现双击缩放
#### 1.1 draw()
```java
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //缩放
        float scale = smallScale + (bigScale - smallScale) * scaleFraction;
        canvas.scale(scale, scale, getWidth() / 2f, getHeight() / 2f);
        //画bitmap
        canvas.drawBitmap(bitmap, originalOffsetX, originalOffsetY, mPaint);
    }
```
#### 1.2 监听双击事件
> 需要先重写onTouchEvent()方法，然后将event直接交给GestureDetector处理。然后我们的逻辑写在GestureDetector中    
#### 1.3 创建动画来控制 scaleFraction 实现过度效果，
```java
    /**
     * 重写触摸监听，并把事件传递给GestureDetectorCompat处理
     *
     * @param event
     * @return
     */
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //把事件交给GestureDetectorCompat，他的onDown事件要返回true，才能真正监听到。
        return mDetectorCompat.onTouchEvent(event);
    }

    /**
     * 双击回调监听
     */
    GestureDetector.OnDoubleTapListener mOnDoubleTapListener = new GestureDetector.OnDoubleTapListener() {
        /**
         * 1、双击，两次点击不超过300ms，
         * 但是连续点击4次，会触发两次。
         * 2、两次点击小于40ms ，会被认为手抖，不会触发
         * @param e
         * @return 没用
         */
        @Override
        public boolean onDoubleTap(MotionEvent e) {
            isBig = !isBig;
            if (isBig) {
                getScaleAnimator().start();
            } else {
                getScaleAnimator().reverse();
            }
            return false;
        }
    };
    
    /**
     * 动画初始化
     *
     * @return
     */
    private ObjectAnimator getScaleAnimator() {
        if (scaleAnimator == null) {
            scaleAnimator = ObjectAnimator.ofFloat(this, "scaleFraction", 0, 1);
        }
        return scaleAnimator;
    }
```

## 2、实现拖动
#### 2.1 translate()
```java
 /**
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //只有在放大的情况再拖动
        if (isBig) {
            //手指拖动
            canvas.translate(transOffsetX, transOffsetY);
        }
        //缩放
        float scale = smallScale + (bigScale - smallScale) * scaleFraction;
        canvas.scale(scale, scale, getWidth() / 2f, getHeight() / 2f);
        //画bitmap
        canvas.drawBitmap(bitmap, originalOffsetX, originalOffsetY, mPaint);
    }
```
#### 2.2 获取移动数值
> 通过**OnGestureListener** 的 **onScroll()** 方法可以监听手势滑动。    
```java
    /**
     * 手势监听回调
     */
    GestureDetector.OnGestureListener mGestureListener = new GestureDetector.OnGestureListener() {
        /**
         * 手指滑动
         * @param down 按下时
         * @param event 移动后
         * @param distanceX 这个x偏移是：lastEvent.x - nowEvent.x = distanceX
         * @param distanceY 同上 也就是他是 起始点 - 终点 所以transOffsetX才用的 -=
         * @return
         */
        @Override
        public boolean onScroll(MotionEvent down, MotionEvent event, float distanceX, float distanceY) {
            //只有放大时可拖动
            if (isBig) {
                transOffsetX -= distanceX;
                //设置不能拖出屏幕
                //x的最大值
                transOffsetX = Math.min(transOffsetX, (bitmap.getWidth() * bigScale - (float) getWidth()) / 2);
                //x的最小值
                transOffsetX = Math.max(transOffsetX, -(bitmap.getWidth() * bigScale - (float) getWidth()) / 2);

                transOffsetY -= distanceY;
                transOffsetY = Math.min(transOffsetY, (bitmap.getHeight() * bigScale - (float) getHeight()) / 2);
                transOffsetY = Math.max(transOffsetY, -(bitmap.getHeight() * bigScale - (float) getHeight()) / 2);
//            Log.e("wzzzzzzzz", "onScroll: " + distanceX + "   " + distanceY + "  " + transOffsetY + "   " + transOffsetY);

                invalidate();
            }
            return false;
        }
    };

```
## 3、实现惯性滑动
#### 3.1 监听触摸事件 GestureDetectorCompat 的 **OnGestureListener** 的 **onFling()** 方法可以监听手势惯性。
#### 3.2 通过 **OverScroller** 惯性计算模型，可以帮助计算惯性值。
```
