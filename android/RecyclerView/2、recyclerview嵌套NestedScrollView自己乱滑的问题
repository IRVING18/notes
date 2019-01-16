## 嵌套之后的问题
### 一、滑动冲突
- 在recyclerView上加上：android:nestedScrollingEnabled="false"
### 二、自己往上窜
- 1.在RecyclerView上加上 android:descendantFocusability="blocksDescendants"
- 2.在NestedScrollView的子view上加上 android:descendantFocusability="blocksDescendants"
- 3.保险起见，去掉recyclerView的焦点获取：android:focusable="false"  android:focusableInTouchMode="false"

```java
<android.support.v4.widget.NestedScrollView
        android:id="@+id/nsv"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:descendantFocusability="blocksDescendants"
            android:orientation="vertical">
            <android.support.v7.widget.RecyclerView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:descendantFocusability="blocksDescendants"
                android:focusable="false"
                android:focusableInTouchMode="false"
                android:nestedScrollingEnabled="false" />
        </LinearLayout>
</android.support.v4.widget.NestedScrollView>
```
