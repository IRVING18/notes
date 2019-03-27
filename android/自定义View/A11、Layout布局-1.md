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
![linear](https://github.com/IRVING18/notes/blob/master/android/file/layoutdemo1.gif)

#### 理解测量顺序：    
> 1、ImageView的onMeasure()方法，设置宽高相等，setMeasureDimensions()。    
> 2、set完之后，父view就能拿到ImageView的宽高，然后会给子view相应的尺寸。

- 1.1 LinearLayout 在onMeasure()方法会调用ImageView的measure()方法，
- 1.2 ImageView的measure()方法会调用onMeasure()，onMeasure(),setMeasureDimensions()设置宽高。
- 1.3 然后LinearLayout通过child.getMeasureWidth()/Height(),来获取ImageView的宽高，然后它会保存下来，然后在onLayout()中去给IamgeView布局。

[详细代码](https://github.com/hencoder/PracticeLayout1)
#### 关键代码
```java
 @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        int measuredWidth = getMeasuredWidth();
        int measuredHeight = getMeasuredHeight();
        if (measuredWidth > measuredHeight) {
            measuredWidth = measuredHeight;
        } else {
            measuredHeight = measuredWidth;
        }
        setMeasuredDimension(measuredWidth, measuredHeight);
    }
```
