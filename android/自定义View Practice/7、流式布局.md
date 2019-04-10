# FlowLayout 流式布局

![linear](https://github.com/IRVING18/notes/blob/master/android/file/flowLayout.jpg)

# 一、实现思路
#### 1、通过重写onMeasure()方法，获取自己的宽高和childView的宽高。
- 1.1 通过调用childView的measure()方法，然后让子view自己测量。这其中涉及MeasureSpec.UNSPECIFIED等模式，但是这些操作都是定式的，只需要调用**measureChildWithMargins()** 方法，就帮我们完成了各个模式下让子view.measure()的方法了。
**注意：这有个坑，调用这个方法时，必须先重写generateLayoutParams方法，然后返回MarginLayoutParams()**
- 1.2 测量完之后，直接可以调用，childView.getMeasureWidth()/Height()来获取childView的宽高。
- 1.3 然后根据childView的宽高，来计算当前childView应该在什么位置，也就是上下左右的坐标，拿到这个数据之后，我们用List<Rect>可以将它存起来，方便onLayout用。 
- 1.4 并根据childView的总高度和最大宽度，可以通过setMeasuredDimension()设置给自己。以便如果自己也在其他布局中的时候，给父view调用。
  
#### 2、通过重写onLayout()方法，来对子View进行布局。把他们的坐标位置通过childView.layout()方法设置给子view
- 2.1 把List<Rect>存储的子view数据，遍历然后通过childView.layout()设置给子view
  
#### 3、具体onMeasure()的计算方法。
- 3.1 通过遍历measureChildWithMargins()测量每一个childview宽高。
- 3.2 通过吧childView的width值加起来lineWidthUsed 对比 当前view的宽度tagLayoutSpecWidthSize = MeasureSpec.getSize(widthMeasureSpec);  
- 3.3 如果lineWidthUsed 小于 tagLayoutSpecWidthSize，那么就直接存到List<Rect>中    
- 3.4 如果lineWidthUsed 大于 tagLayoutSpecWidthSize，那么就错一行，把高度加高，然后把新高度当成起始点。这时需要再调用measureChildWithMargins一次，因为上一次调用时，如果当前view的宽度不够它全展开，那么它会自己压缩自己，所以我们需要把lineWidthUsed设为0，再调一次。

# 二、实现代码

关键代码  

```java
/**
     * 测量自己的高度和子view的高度
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        //横向当前行已使用的宽度
        int lineWidthUsed  = 0;
        //总高度
        int allheightUsed = 0;
        //所有行最高和最宽的值 , 可以作为tagView 的宽
        int maxWidth  = 0;
        int maxHeight  = 0;
        //获取当前layoutview的mode和宽度
        int tagLayoutSpecWidthMode = MeasureSpec.getMode(widthMeasureSpec);
        int tagLayoutSpecWidthSize = MeasureSpec.getSize(widthMeasureSpec);

        for (int i = 0; i < getChildCount(); i++) {
            View child = getChildAt(i);

            //1、方法作用：测量子view
            //2、注意使用时需要重写generateLayoutParams方法，返回一个MarginLayoutParams
            measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, allheightUsed);
            //1、UNSPECIFIED 的话就永远不用重新测绘childview，因为父view的宽度没有限制（处在horizonScrollView），那子view直接一直往后排就好了。
            //2、如果已使用的宽度 + 当前测量的child宽度 > 当前Layout的宽度的话，就错行，再重新量一遍当前正在量的child。
            //再量一次当前的childview是因为，上次测量如果父view宽度就不够，他就会压缩自己，这样就不准确了。
            if (tagLayoutSpecWidthMode != MeasureSpec.UNSPECIFIED && lineWidthUsed + child.getMeasuredWidth() > tagLayoutSpecWidthSize) {
                //当前行 置为头部
                lineWidthUsed = 0;
                allheightUsed += maxHeight;
                maxHeight = 0;
                measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, allheightUsed);
            }

            //拿到自view的尺寸，存起来
            Rect childBound;
            if (childrenBounds.size() <= i) {
                childBound = new Rect();
                childrenBounds.add(childBound);
            } else {
                childBound = childrenBounds.get(i);
            }
            //存储当前childview的位置
            childBound.set(lineWidthUsed, allheightUsed, lineWidthUsed + child.getMeasuredWidth(), allheightUsed + child.getMeasuredHeight());

            lineWidthUsed += child.getMeasuredWidth();
            //存储最宽的值
            maxWidth = Math.max(lineWidthUsed, maxWidth);
            maxHeight = Math.max(maxHeight, child.getMeasuredHeight());
        }
        //保存当前view的尺寸
        int width  = maxWidth;
        int height = allheightUsed + maxHeight;
        setMeasuredDimension(width, height);

    }

    /**
     * 给子view去布局
     */
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        for (int i = 0; i < getChildCount(); i++) {
            View child = getChildAt(i);
            Rect rect  = childrenBounds.get(i);
            child.layout(rect.left, rect.top, rect.right, rect.bottom);
        }
    }

    @Override
    public LayoutParams generateLayoutParams(AttributeSet attrs) {
        return new MarginLayoutParams(getContext(), attrs);
    }
```
