# MaterialEditText

![linear](https://github.com/rengwuxian/MaterialEditText/blob/master/images/floating_label.png)
## 一、动画分析
> 方法：可以解开动图然后观察
- 文字一个简单的渐变alpha 0->1。
- y轴坐标的移动。
- 唯一的关键就是设置padding，来让顶部腾出位置来绘制文字。
## 二、实现思路
#### 设置一个fraction值，用来同时控制alpha和y轴坐标的渐变。
1. alpha = 255 * fraction;
2. y = baseLine + (1 - fraction) * MoveValue;
#### 设置padding
```java
    //获取背景的padding，但是设置setPadding()方法设置的view的padding。所以背景的padding不会变。
    //这样就可方便将padding设置为原值，主要用来动态设置是否显示顶部文字的。
    private Rect backgroundPadding;
    getBackground().getPadding(backgroundPadding);
    int paddingTop;
    paddingTop = (int) (backgroundPadding.top + TEXT_SIZE + TEXT_MARGIN);
    //设置view的Padding
    setPadding(getPaddingLeft(), paddingTop, getPaddingRight(), getPaddingBottom());
```

## 三、实现代码

[具体项目地址](https://github.com/IRVING18/MaterialEditText)

关键代码：

```java
       public static final float          TEXT_SIZE   = DisplayUtils.dip2px(14);
    public static final float          TEXT_MARGIN = DisplayUtils.dip2px(10);
    private             Paint          mPaint;
    private             Rect           backgroundPadding;
    private             Rect           rectTextBounds;
    private             boolean        showTopTag  = false;
    private             float          fraction;
    private             ObjectAnimator mAnimator;
    private             boolean        useLabel    = true;
    /**
     * 设置顶部label显示位置
     */
    private void setBackgroundPadding() {
        if (rectTextBounds == null)
            rectTextBounds = new Rect();
        //获取背景的padding，但是设置padding设置的view的padding，而背景的padding不会变。主要用来动态设置是否显示顶部文字的。
        if (backgroundPadding == null)
            backgroundPadding = new Rect();
        getBackground().getPadding(backgroundPadding);
        int paddingTop;
        if (useLabel) {
            paddingTop = (int) (backgroundPadding.top + TEXT_SIZE + TEXT_MARGIN);
        } else {
            paddingTop = backgroundPadding.top;
        }
        //设置view的Padding
        setPadding(getPaddingLeft(), paddingTop, getPaddingRight(), getPaddingBottom());
    }

    /**
     * 获取动画
     *
     * @return
     */
    private ObjectAnimator getAnimator() {
        if (mAnimator == null) {
            mAnimator = ObjectAnimator.ofFloat(this, "fraction", 0, 1);
        }

        return mAnimator;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //画顶部文字
        String hint = getHint().toString();
        mPaint.setAlpha((int) (fraction * 0xff));//0xff 就是255
        mPaint.getTextBounds(hint, 0, hint.length(), rectTextBounds);
        int y = (int) (backgroundPadding.top + TEXT_SIZE + TEXT_MARGIN) / 2 - (rectTextBounds.top + rectTextBounds.bottom) / 2;
        canvas.drawText(getHint(), 0, getHint().length(), getPaddingLeft(), y + (getHeight() - y) * (1 - fraction), mPaint);
    }

```

