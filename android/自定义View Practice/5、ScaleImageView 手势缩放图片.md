# ScaleIamgeView 手势缩放图片

## 一、获取当前view的宽高
> 1、在layout的时候，当前view会被父类调用，并拿到自己的属性。所以可以在这拿到view宽高。   
> 2、onLayout()方法是layout调度的，当然也能拿到。

```java
layout();
onlayout();
onSizeChanged();
```
## 二、GestureDetectorCompat 触摸手势识别
### 1、普通手势监听 GestureDetector.OnGestureListener
- 1.1 onDown() 所有方法是否能收到回调的**根本**。返回true能收到，false收不到
- 1.2 onShowPress(MotionEvent e) 预按下事件，按下100ms回调。
- 1.3 onSingleTapUp(MotionEvent e) 按下抬起就触发，单击。但是同时监听双击事件的话，那么单击功能就要用**onSingleTapConfirmed()** 方法了。
- 1.4 onScroll(MotionEvent down, MotionEvent event, float distanceX, float distanceY) 滑动回调，**注意distanceX和Y是 初始点 - 终点**
- 1.5 onLongPress(MotionEvent e)长按回调，按下500ms就会回调
- 1.6 onFling(MotionEvent down, MotionEvent event, float velocityX, float velocityY)惯性滑动监听，**通常配合OverScroller使用** 。
```java
    /**
     * 手势监听回调
     */
    GestureDetector.OnGestureListener mGestureListener = new GestureDetector.OnGestureListener() {
        /**
         * 确定是否消费事件
         * @return true :消费，false：不消费
         * 不消费的话其他回调都不会收到
         */
        @Override
        public boolean onDown(MotionEvent e) {
            return true;
        }

        /**
         * 预按下，100ms 到达
         * todo:上一期的预按下回顾
         * @param e
         */
        @Override
        public void onShowPress(MotionEvent e) {
        }

        /**
         * 按下抬起，单机，<p>
         * 1、默认长按开启的话，超过500ms再抬起就不会收到回调了。
         * 2、如果设置关闭长按，那么就没有500ms限制了，随时抬起都会有回调。，mDetectorCompat.setIsLongpressEnabled(false);
         * @param e
         * @return 返回值没有用
         */
        @Override
        public boolean onSingleTapUp(MotionEvent e) {
            return false;
        }

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
            return false;
        }

        /**
         * 长按回调，500ms
         * @param e
         */
        @Override
        public void onLongPress(MotionEvent e) {
        }

        /**
         * 惯性滑动
         * @param velocityX 惯性越大，值越大
         * @return 返回值没有
         */
        @Override
        public boolean onFling(MotionEvent down, MotionEvent event, float velocityX, float velocityY) {
            return false;
        }
    };
```

### 2、GestureDetector.OnDoubleTapListener 双击监听
