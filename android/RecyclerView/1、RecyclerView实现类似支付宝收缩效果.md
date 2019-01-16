## RecyclerView实现支付宝自动收缩效果
### 一、思路
- 1.首先实现渐变Toolbar效果，通过recyclerView监听滑动。先让上边的Head1的alpha从1到0，然后在让Head2的alpha从0到1.
- 2.自动收缩，在中间值划分，小于中间值收缩到0，大于中间值扩展到最大值。
- 3.收缩功能实现：使用recyclerview.smoothScrollBy(dx,dy,InInterpolator);    
  > dx dy:代表的是相对当前位置xy轴滑动的距离，而不是scrollTo的功能。而且recyclerview的scrollTo功能没有系统实现代码，调用也不好使。
  
- 4.如果解决当recyclerview就不够扩展的，怎么办？   
  > recyclerView.computeVerticalScrollRange();获取未被回收的item的总体高度。   
  > 退而求其次，只能用这个货来鉴定了，如果这个货 > 收缩的高度 + recyclerView控件的固定显示高度，就让这个功能生效，否则不生效。    
  > 实践起来好像还可以用，只要收缩高度不要太夸张就好。   
  
- 5.获取收缩高度。   
  > topHeight获取的话，通过linearlayoutManager获取item，然后获取高度。   
  > 1.但是需要在recyclerview的layout()方法之后。所以我直接放在了，scroll监听中，只测量一次。   
  > 2.操作getChildAt()很容易throwException，因为recyclerview回收之后就容易出问题，所以try一下。
### 二、获取topHeight关键代码
```java
/**
     * 获取topHeight
     */
    private void setTopHeight() {
        try {
            //只获取一次，在没有值得时候获取
            if (topHeight == 0) {
                List<HomeRyVo> datas = mRecyclerViewAdapter.getDatas();
                //数组至少需要三个才能算
                if (datas.size() > 2
                        //第一个必须是会员模块
                        && datas.get(0).getItem_type().equals(HomeRecyclerAdapter.TYPE_MEMBER + "")
                        //第二个可以是跑马灯或者sdjg
                        && (datas.get(1).getItem_type().equals(HomeRecyclerAdapter.TYPE_SDJG + "") || datas.get(1).getItem_type().equals(HomeRecyclerAdapter.TYPE_MARQUEE + ""))
                        //第三个可以是跑马灯或者sdjg
                        && (datas.get(2).getItem_type().equals(HomeRecyclerAdapter.TYPE_SDJG + "") || datas.get(2).getItem_type().equals(HomeRecyclerAdapter.TYPE_MARQUEE + ""))) {
                    LinearLayoutManager layoutManager = (LinearLayoutManager) mRecyclerView.getLayoutManager();
                    View childAt0 = layoutManager.getChildAt(0);
                    int h0 = childAt0.getMeasuredHeight();
                    View childAt1 = layoutManager.getChildAt(1);
                    int h1 = childAt1.getMeasuredHeight();
                    View childAt2 = layoutManager.getChildAt(2);
                    int h2 = childAt2.getMeasuredHeight();
                    topHeight = h0 + h1 + h2 - mRlToolbar.getMeasuredHeight();
                } else {
                    topHeight = AbDisplayUtil.getScreenHeight() / 4;
                }
            }
        } catch (Exception e) {
            topHeight = AbDisplayUtil.getScreenHeight() / 4;
        }

    }

```
### 三、收缩功能关键代码
```java
mRecyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {
            private float halfHeight;
            private int scrollY = 0;

            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                super.onScrolled(recyclerView, dx, dy);
                //获取topHeight
                setTopHeight();
                //scrollY 就是垂直偏移量
                scrollY += dy;
                if (scrollY <= 0) {
                    mRlToolbar.setBackgroundColor(Color.TRANSPARENT);
                } else if (scrollY <= topHeight) {
                    float alpha = (float) scrollY / topHeight;
                    mStatusBarColor = ColorUtils.blendARGB(Color.TRANSPARENT
                            , toolbarEndColor, alpha);
                    mRlToolbar.setBackgroundColor(mStatusBarColor);
                } else {
                    mStatusBarColor = ColorUtils.blendARGB(Color.TRANSPARENT
                            , toolbarEndColor, 1);
                    mRlToolbar.setBackgroundColor(mStatusBarColor);
                }

                halfHeight = topHeight / 2;
                if (scrollY <= 0) {
                    mRlHead1.setAlpha(1);
                    mRlHead2.setAlpha(0);
                } else if (scrollY <= halfHeight) {
                    float al = 1 - (scrollY / halfHeight);
                    mRlHead1.setAlpha(al);
                } else if (scrollY < topHeight) {
                    mRlHead2.setAlpha((scrollY - halfHeight) / halfHeight);
                } else {
                    mRlHead1.setAlpha(0);
                    mRlHead2.setAlpha(1);
                }
            }

            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);

//                AbLazyLogger.e("wzzzzzzz IDLE " + mRecyclerView.computeVerticalScrollRange());
                //获取recyclerview未回收的item的总高度，这个高度如果大于自动向上滑动的距离加上recyclerview控件的高度，再提供自动收缩功能。
                if (mRecyclerView.computeVerticalScrollRange() > topHeight + mRecyclerView.getMeasuredHeight()) {
                    //IDLE是自动滑动结束后的滚动距离
                    if (newState == RecyclerView.SCROLL_STATE_IDLE) {
//                        AbLazyLogger.e("wzzzzzzz IDLE " + scrollY + "   " + halfHeight + "  " + topHeight);
                        if (scrollY < 0) {

                        } else if (scrollY < halfHeight) {
                            //smoothScrollBy的dy:是相对当前高度的滑动距离。
                            mRecyclerView.smoothScrollBy(0, -scrollY, new FastOutSlowInInterpolator());
                        } else if (scrollY < topHeight) {
                            mRecyclerView.smoothScrollBy(0, topHeight - scrollY, new FastOutSlowInInterpolator());
                        }
                    }
                }

//                else if (newState == RecyclerView.SCROLL_STATE_DRAGGING) {
//                    AbLazyLogger.e("wzzzzzzz DRAGGING " + scrollY);
//                } else if (newState == RecyclerView.SCROLL_STATE_SETTLING) {
//                    AbLazyLogger.e("wzzzzzzz SETTLING " + scrollY);
//                }

            }
        });
```

