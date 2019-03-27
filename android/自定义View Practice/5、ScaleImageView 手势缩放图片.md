# ScaleIamgeView 手势缩放图片

## 获取当前view的宽高
> 1、在layout的时候，当前view会被父类调用，并拿到自己的属性。所以可以在这拿到view宽高。   
> 2、onLayout()方法是layout调度的，当然也能拿到。

```java
layout();
onlayout();
onSizeChanged();
```
