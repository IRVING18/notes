# 1、PathMeasure
> 测量path的长度。   
> 使用场景画表盘，可以可以使用pathMearsure.getLength()来获取弧形的长度，然后再等分画刻度。
# Xformode 使用
> 用于合成图片，两张图片交汇怎么处理的。

[具体文档](https://hencoder.com/ui-1-2/)

![linear](https://ws3.sinaimg.cn/large/52eb2279ly1fig6im3hhcj20o50zt7bj.jpg)

### 实例1
> 效果图

![linear](https://github.com/IRVING18/notes/blob/master/android/file/circle.png)

#### 实现思路
- 1、先画一个大圆设置成绿色。
- 2、画一个合成后的图片，将圆形和头像合成的图片。
- 3、实现合成的图片，需要使用离屏缓冲，就是将某一块拿出来画好再填回去，因为在使用xformode的时候不设置离屏缓冲不生效。
- 4、实现xformode组合，先画一个圆形框，在设置xformode，再画头像。

#### 关键代码
```java
 public static final float WIDTH   = DisplayUtils.dip2px(300);
    public static final float PADDING = DisplayUtils.dip2px(20);
    public static final float Large = DisplayUtils.dip2px(10);

    RectF circleRectF = new RectF(PADDING, PADDING, WIDTH + PADDING, WIDTH + PADDING);
    RectF circleRectFLarger = new RectF(PADDING-Large, PADDING-Large, WIDTH + PADDING+Large, WIDTH + PADDING+Large);

    Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);

    Xfermode xfermode = new PorterDuffXfermode(PorterDuff.Mode.SRC_IN);

    Bitmap bitmap;

    public AvatarXformodeView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);

    }

    {
        bitmap = getAvator((int) WIDTH, R.drawable.what_the_fuck);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        //画外部绿色框
        paint.setColor(Color.GREEN);
        canvas.drawOval(circleRectFLarger,paint);
        paint.reset();

        //设置circleRectF范围离屏缓冲
        int save = canvas.saveLayer(circleRectF, paint);
        //画圆形框
        canvas.drawOval(circleRectF, paint);
        paint.setXfermode(xfermode);
        //画图形
        canvas.drawBitmap(bitmap, PADDING, PADDING, paint);
        //恢复离屏缓冲，就是把图形绘制完再绘制到画布上
        paint.setXfermode(null);
        canvas.restoreToCount(save);

    }

    public Bitmap getAvator(int width, @DrawableRes int res) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        //只测绘宽高
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeResource(getResources(), res, options);
        options.inJustDecodeBounds = false;
        options.inDensity = options.outWidth;
        options.inTargetDensity = width;
        return BitmapFactory.decodeResource(getResources(), res, options);

    }
```
