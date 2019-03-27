# ScaleIamgeView 手势缩放图片

[基础知识](https://github.com/IRVING18/notes/blob/master/android/自定义View/A12、手势触摸-初探.md)

# 一、实现思路
#### 1、实现双击缩放 
- 1.1 canvas.**scale()** 来实现 
- 1.2 监听双击事件 GestureDetectorCompat 的 **OnDoubleTapListener** 的 **onDoubleTapEvent()**方法
#### 2、实现拖动
- 2.1 canvas.**translate()** 来实现
- 2.2 监听触摸事件 GestureDetectorCompat 的 **OnGestureListener** 的 **onScroll()** 方法可以监听手势滑动。
#### 3、实现惯性滑动
- 3.1 监听触摸事件 GestureDetectorCompat 的 **OnGestureListener** 的 **onFling()** 方法可以监听手势惯性。
- 3.2 通过 **OverScroller** 惯性计算模型，可以帮助计算惯性值。



