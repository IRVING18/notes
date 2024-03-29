# 目录
- [一、估值器 TypeEvaluator](#jump1)
  - [（1）系统自带估值器](#jump1_1)
  - [（2）自定义估值器](#jump1_2)
    - [1.ProVinceEvaluator 城市字段String类型估值器](#jump1_2_1)
    - [2.CubicBezierPointFTypeEvaluator 贝塞尔曲线](#jump1_2_2)
- [二、插值器 Interpolator](#jump2)
  - [（1）系统自带插值器 & xml/java使用方式](#jump2_1)
    - [1.AccelerateDecelerateInterpolator 在动画开始与结束的地方速率改变比较慢，在中间的时候加速](#jump2_1_1)
    - [2.AccelerateInterpolator 在动画开始的地方速率改变比较慢，然后开始加速](#jump2_1_2)
    - [3.AnticipateInterpolator 开始的时候向后然后向前甩](#jump2_1_3)
    - [4.AnticipateOvershootInterpolator 开始的时候向后然后向前甩一定值后返回最后的值](#jump2_1_4)
    - [5.BounceInterpolator 动画结束的时候弹起](#jump2_1_5)
    - [6.CycleInterpolator 动画循环播放特定的次数，速率改变沿着正弦曲线](#jump2_1_6)
    - [7.DecelerateInterpolator 在动画开始的地方快然后慢](#jump2_1_7)
    - [8.LinearInterpolator 以常量速率改变](#jump2_1_8)
    - [9.OvershootInterpolator 向前甩一定值后再回到原来位置](#jump2_1_9)
  - [（2）自定义插值器](#jump2_2)
  - [（3）插值器的辅助妙用](#jump2_3)
  
  

- [文中部分图片来源:Carson带你学安卓](https://www.jianshu.com/p/2f19fe1e3ca1)

# <p id="jump1" />一、TypeEvaluator 估值器
> 估值器，用来计算动画过程中的值

## <p id="jump1_1" />（1）系统自带估值器
![](https://github.com/IRVING18/notes/blob/master/android/file/commonTypeEvaluator.png)

### 1.ArgbEvaluator 来做颜色渐变的动画
**例子1：**
> 使用系统自带的ArgbEvaluator
```java
ObjectAnimator animator = ObjectAnimator.ofInt(view, "color", 0xffff0000, 0xff00ff00);  
animator.setEvaluator(new ArgbEvaluator());  
animator.start();  
```
![](https://github.com/IRVING18/notes/blob/master/android/file/argb.gif)

**例子2：**
> 自建HSV模式HsvEvaluator。
[点这里](https://www.baidu.com)
 
### 2.PointFEvaluator 来自定义其他Evaluator 计算器
> 自定义Evaluator 设置    
```java
    private class PointFEvaluator implements TypeEvaluator<PointF> {
        PointF outPointF = new PointF();
        // 重写 evaluate() 方法，让 PointF 可以作为属性来做动画
        @Override
        public PointF evaluate(float fraction, PointF startValue, PointF endValue) {
            outPointF.x = startValue.x + (endValue.x - startValue.x) * fraction;
            outPointF.y = startValue.y + (endValue.y - startValue.y) * fraction;
            return outPointF;
        }
    }
```
> 配合ObjectAnimator使用    
```java
    ObjectAnimator animator = ObjectAnimator.ofObject(view, "position",new PointFEvaluator(), new PointF(0, 0), new PointF(1, 1));
    animator.setInterpolator(new LinearInterpolator());
    animator.setDuration(1000);
    animator.start();
```
## <p id="jump1_2" />（2）自定义估值器
### <p id="jump1_2_1" />1.ProVinceEvaluator 城市字段String类型估值器
> 自定义Evaluator 设置城市String类型参数动画改变    
```java
    private class ProVinceEvaluator implements TypeEvaluator<String> {
        // 重写 evaluate() 方法，让 String 可以作为属性来做动画
        @Override
        public String evaluate(float fraction, String startValue, String endValue) {
            //北京市  ->    上海  中间所有的城市列表list，对list做操作
            int startIndex = proList.indexOf(startValue);
            int endIndex = proList.indexOf(endValue);
            int index = startIndex + (endIndex - startIndex) * fraction;
            
            return proList.get(index);
        }
    }
```
> 配合ObjectAnimator使用    
```java
    ObjectAnimator animator = ObjectAnimator.ofObject(view, "province",new ProVinceEvaluator(), "北京市","上海市");
    animator.setInterpolator(new LinearInterpolator());
    animator.setDuration(1000);
    animator.start();
```
### <p id="jump1_2_2" />2.CubicBezierPointFTypeEvaluator 贝塞尔曲线

[项目地址](https://github.com/IRVING18/DialogAnimDemo)

<img src="https://github.com/IRVING18/notes/blob/master/android/file/cubic.gif" alt="效果图" width="150" height="300" />
```java
    public class CubicBezierPointFTypeEvaluator implements TypeEvaluator<PointF> {
    /**
     * 每个估值器对应一个属性动画，每个属性动画仅对应唯一一个控制点
     */
    PointF control;
    /**
     * 估值器返回值
     */
    PointF mPointF = new PointF();

    public CubicBezierPointFTypeEvaluator(PointF control) {
        this.control = control;
    }

    @Override
    public PointF evaluate(float fraction, PointF startValue, PointF endValue) {
        return getBezierPoint(startValue, endValue, control, fraction);
    }

    /**
     * 二次贝塞尔曲线公式
     *
     * @param start   开始的数据点
     * @param end     结束的数据点
     * @param control 控制点
     * @param t       float 0-1
     * @return 不同t对应的PointF
     */
    private PointF getBezierPoint(PointF start, PointF end, PointF control, float t) {
        mPointF.x = (1 - t) * (1 - t) * start.x + 2 * t * (1 - t) * control.x + t * t * end.x;
        mPointF.y = (1 - t) * (1 - t) * start.y + 2 * t * (1 - t) * control.y + t * t * end.y;
        return mPointF;
    }
}
```
![](https://github.com/IRVING18/notes/blob/master/android/file/cubic-bezier.gif)


[贝塞曲线：二次、三次模拟网站](https://cubic-bezier.com/#.01,1.53,0,1.54)
> 配合ObjectAnimator使用   
```java
private PointF mPointF;

public void setMPointF(PointF pointF) {
    this.mPointF = pointF;
    linear_background.setTranslationX(mPointF.x);
    linear_background.setTranslationY(mPointF.y);
}

private void setExitAnimation(View view) {
    //起点
    PointF startP = new PointF();
    startP.x = 0;
    startP.y = 0;
    //终点
    PointF endP = new PointF();
    endP.x = - 500;
    endP.y = - 800;
    //控制点位置
    PointF controlP = new PointF();
    controlP.x = startP.x;
    controlP.y = -1200;
    //通过估值器计算曲线坐标
    ObjectAnimator anim = ObjectAnimator.ofObject(this, "mPointF", new CubicBezierPointFTypeEvaluator(controlP),startP,endP );
    anim.setDuration(800);
}
```

# <p id="jump2" />二、插值器 Interpolator
> 插值器，用来控制动画过程速率、偏移量的

## <p id="jump2_1" />（1）系统自带插值器
使用方式

```java
/*
 * 使用方式1：xml
 * 主要是设置插值器属性 android:interpolator
 */
 <?xml version="1.0" encoding="utf-8"?>
    <scale xmlns:android="http://schemas.android.com/apk/res/android"

        // 通过资源ID设置插值器
        android:interpolator="@android:anim/overshoot_interpolator"
        android:duration="3000"
        android:fromXScale="0.0"
        android:fromYScale="0.0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="2"
        android:toYScale="2" />
 >

/*
 * 使用方式2：java
 */
// 步骤1:创建 需要设置动画的 视图View
Button mButton = (Button) findViewById(R.id.Button);
// 步骤2：创建透明度动画的对象 & 设置动画效果
Animation alphaAnimation = new AlphaAnimation(1,0);
// 步骤3：创建对应的插值器类对象
alphaAnimation.setDuration(3000);
Interpolator overshootInterpolator = new OvershootInterpolator();
// 步骤4：给动画设置插值器
alphaAnimation.setInterpolator(overshootInterpolator);
// 步骤5：播放动画
mButton.startAnimation(alphaAnimation);
```

### <p id="jump2_1_1"/>1.AccelerateDecelerateInterpolator 在动画开始与结束的地方速率改变比较慢，在中间的时候加速
### <p id="jump2_1_2"/>2.AccelerateInterpolator 在动画开始的地方速率改变比较慢，然后开始加速
### <p id="jump2_1_3"/>3.AnticipateInterpolator 开始的时候向后然后向前甩
### <p id="jump2_1_4"/>4.AnticipateOvershootInterpolator 开始的时候向后然后向前甩一定值后返回最后的值
### <p id="jump2_1_5"/>5.BounceInterpolator 动画结束的时候弹起
### <p id="jump2_1_6"/>6.CycleInterpolator 动画循环播放特定的次数，速率改变沿着正弦曲线
### <p id="jump2_1_7"/>7.DecelerateInterpolator 在动画开始的地方快然后慢
### <p id="jump2_1_8"/>8.LinearInterpolator 以常量速率改变
### <p id="jump2_1_9"/>9.OvershootInterpolator 向前甩一定值后再回到原来位置

![](https://github.com/IRVING18/notes/blob/master/android/file/commonInterpolator.png)

演示：

![](https://github.com/IRVING18/notes/blob/master/android/file/interpolator.gif)

## <p id="jump2_2" />（2）自定义插值器
> 根据动画的进度（0%-100%）计算出当前属性值改变的百分比

### 1、实现方式： Interpolator 或 TimeInterpolator 接口
> 1、补间动画 实现 Interpolator接口；属性动画实现TimeInterpolator接口
> 
> 2、TimeInterpolator接口是属性动画中新增的，用于兼容Interpolator接口，这使得所有过去的Interpolator实现类都可以直接在属性动画使用


```java
// Interpolator接口
public interface Interpolator {  

    // 内部只有一个方法：getInterpolation()
     float getInterpolation(float input) {  
        // 参数说明
        // input值值变化范围是0-1，且随着动画进度（0% - 100% ）均匀变化
        // 即动画开始时，input值 = 0；动画结束时input = 1
        // 而中间的值则是随着动画的进度（0% - 100%）在0到1之间均匀增加
        
      ...// 插值器的计算逻辑

      return xxx；
      // 返回的值就是用于估值器继续计算的fraction值，
    }  

// TimeInterpolator接口
// 同上
public interface TimeInterpolator {  
  
    float getInterpolation(float input){
         ...
    };  
}  
```

看两个系统的插值器源码就理解参数意义了

```java
/*
 * 匀速差值器：LinearInterpolator
 */
@HasNativeInterpolator  
public class LinearInterpolator extends BaseInterpolator implements NativeInterpolatorFactory {  
   
    ... // 仅贴出关键代码

    public float getInterpolation(float input) {  
        return input;  
        // 没有对input值进行任何逻辑处理，直接返回
        // 即input值 = fraction值
        // 因为input值是匀速增加的，因此fraction值也是匀速增加的，所以动画的运动情况也是匀速的，所以是匀速插值器
    }  

/*
 * 先加速再减速 差值器:AccelerateDecelerateInterpolator
 */
@HasNativeInterpolator  
public class AccelerateDecelerateInterpolator implements Interpolator, NativeInterpolatorFactory {  
      
    ... // 仅贴出关键代码
    
    public float getInterpolation(float input) {  
        return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;
        // input的运算逻辑如下：
        // 使用了余弦函数，因input的取值范围是0到1，那么cos函数中的取值范围就是π到2π。
        // 而cos(π)的结果是-1，cos(2π)的结果是1
        // 所以该值除以2加上0.5后，getInterpolation()方法最终返回的结果值还是在0到1之间。只不过经过了余弦运算之后，最终的结果不再是匀速增加的了，而是经历了一个先加速后减速的过程
        // 所以最终，fraction值 = 运算后的值 = 先加速后减速
        // 所以该差值器是先加速再减速的
    }  
}
```

## <p id="jump2_3" />（3）插值器的辅助妙用
## 用法一：
> Interpolator插值器的用法，getInterpolation(percent百分比)能获取插值器相关参数

根据interpolation.getInterpolation(percent)方法可以直接控制动画的相关属性速率。   

**场景一： 画圈圈**   

1. canvas.drawCircle()就可以   
2. 怎么控制画自定多个circle画呢，可以用list存起来，然后遍历list 然后draw   
3. 怎么控制circle取消，只需要给个边界值，超出边界的circle就remove掉。这里使用的是以时间2s做的边界值。   

实现代码

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


