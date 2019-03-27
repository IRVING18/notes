# Layout布局

## 一、方法说明和流程

<img width="1000" height=“593” src="https://github.com/IRVING18/notes/blob/master/android/file/layout1.png"></img>

## 二、获取当前view的宽高
> 1、在layout的时候，当前view会被父类调用，并拿到自己的属性。所以可以在这拿到view宽高。   
> 2、onLayout()方法是layout调度的，当然也能拿到。

```java
layout();
onlayout();
onSizeChanged();
```

## 三、实例说明
### 1、正方形的ImageView
