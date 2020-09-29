+++
title= "ScrollView嵌套ListView显示不全的解决方法"
date= "2017-09-22"
categories= [ "Android" ]
+++
摘要:开发中经常遇到ScrollView嵌套ListView,GridView,或者RecyclerView嵌套RecyclerView的情况,常常会出现显示不全的现象,下面提供几种不同的解决方法

1.在不是很复杂的布局的情况下,尽量不嵌套,使用添加头布局尾布局的方式进行实现;RecyclerView的添加方式可以参考张鸿洋的博客

2.网上比较流行的是自定义ListView和RecyclerView的LayoutManager
(1)自定义ListView,GridView,重写onMeasure方法
ListView:

```
public class MyListView extends ListView {  
    public MyListView(Context context, AttributeSet attrs) {  
        super(context, attrs);  
    }  
  
    public void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
        int mExpandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);  
        super.onMeasure(widthMeasureSpec, mExpandSpec);  
    }  
  
}  
```
这种方法是将ListView的允许高度设置为极大值,比较简单,但是有一个后果是会造成adapter的getView方法调用多次,也就是多次重绘计算高度.导致的结果就是加载卡顿,如果item中有EditText的话可能会造成数据显示混乱
GridView也类似:

```
public class MyGridView extends GridView {  
    public MyGridView(Context context) {  
        super(context);  
    }  
  
    public MyGridView(Context context, AttributeSet attrs) {  
        super(context, attrs);  
    }  
  
    public MyGridView(Context context, AttributeSet attrs, int defStyleAttr) {  
        super(context, attrs, defStyleAttr);  
    }  
  
    @Override  
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE>>2,MeasureSpec.AT_MOST);  
        super.onMeasure(widthMeasureSpec,expandSpec);  
    }  
}  
```
(2)自定义RecyclerView的LayoutManager
LinearLayoutManager:

```
public class MyLinearLayoutManager extends LinearLayoutManager {  
  
    private static final String TAG = MyLinearLayoutManager.class.getSimpleName();  
  
    public MyLinearLayoutManager(Context context) {  
        super(context);  
    }  
  
    public MyLinearLayoutManager(Context context, int orientation, boolean reverseLayout) {  
        super(context, orientation, reverseLayout);  
    }  
  
    private int[] mMeasuredDimension = new int[2];  
  
    @Override  
    public void onMeasure(RecyclerView.Recycler recycler, RecyclerView.State state,  
                          int widthSpec, int heightSpec) {  
  
        final int widthMode = View.MeasureSpec.getMode(widthSpec);  
        final int heightMode = View.MeasureSpec.getMode(heightSpec);  
        final int widthSize = View.MeasureSpec.getSize(widthSpec);  
        final int heightSize = View.MeasureSpec.getSize(heightSpec);  
        int width = 0;  
        int height = 0;  
        for (int i = 0; i < getItemCount(); i++) {  
            measureScrapChild(recycler, i,  
                    View.MeasureSpec.makeMeasureSpec(i, View.MeasureSpec.UNSPECIFIED),  
                    View.MeasureSpec.makeMeasureSpec(i, View.MeasureSpec.UNSPECIFIED),  
                    mMeasuredDimension);  
  
            if (getOrientation() == HORIZONTAL) {  
                width = width + mMeasuredDimension[0];  
                if (i == 0) {  
                    height = mMeasuredDimension[1];  
                }  
            } else {  
                height = height + mMeasuredDimension[1];  
                if (i == 0) {  
                    width = mMeasuredDimension[0];  
                }  
            }  
        }  
        switch (widthMode) {  
            case View.MeasureSpec.EXACTLY:  
                width = widthSize;  
            case View.MeasureSpec.AT_MOST:  
            case View.MeasureSpec.UNSPECIFIED:  
        }  
  
        switch (heightMode) {  
            case View.MeasureSpec.EXACTLY:  
                height = heightSize;  
            case View.MeasureSpec.AT_MOST:  
            case View.MeasureSpec.UNSPECIFIED:  
        }  
  
        setMeasuredDimension(width, height);  
    }  
  
    private void measureScrapChild(RecyclerView.Recycler recycler, int position, int widthSpec,  
                                   int heightSpec, int[] measuredDimension) {  
        try {  
            View view = recycler.getViewForPosition(0);  
  
            if (view != null) {  
                RecyclerView.LayoutParams p = (RecyclerView.LayoutParams) view.getLayoutParams();  
  
                int childWidthSpec = ViewGroup.getChildMeasureSpec(widthSpec,  
                        getPaddingLeft() + getPaddingRight(), p.width);  
  
                int childHeightSpec = ViewGroup.getChildMeasureSpec(heightSpec,  
                        getPaddingTop() + getPaddingBottom(), p.height);  
  
                view.measure(childWidthSpec, childHeightSpec);  
                measuredDimension[0] = view.getMeasuredWidth() + p.leftMargin + p.rightMargin;  
                measuredDimension[1] = view.getMeasuredHeight() + p.bottomMargin + p.topMargin;  
                recycler.recycleView(view);  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
        } finally {  
        }  
    }  
}  

GridLayoutManager:**[java]** [view plain](http://blog.csdn.net/kinly1993/article/details/54616348#) [copy](http://blog.csdn.net/kinly1993/article/details/54616348#)

public class MyGridLayoutManager extends GridLayoutManager {  
  
    private Context context;  
      
    public MyGridLayoutManager(Context context, int spanCount) {  
        super(context, spanCount);  
        this.context = context;  
    }  
  
    public MyGridLayoutManager(Context context, int spanCount,  
                               int orientation, boolean reverseLayout) {  
        super(context, spanCount, orientation, reverseLayout);  
        this.context = context;  
    }  
  
    private int[] mMeasuredDimension = new int[2];  
  
    @SuppressLint("NewApi")  
    @Override  
    public void onMeasure(RecyclerView.Recycler recycler, RecyclerView.State state, int widthSpec, int heightSpec) {  
        final int heightMode = View.MeasureSpec.getMode(heightSpec);  
        final int widthSize = View.MeasureSpec.getSize(widthSpec);  
        final int heightSize = View.MeasureSpec.getSize(heightSpec);  
  
        int height = 0;  
        int count = getItemCount();  
        int span = getSpanCount();  
          
        for (int i = 0; i < count; i++) {  
            measureScrapChild(recycler, i, View.MeasureSpec.makeMeasureSpec(i,  
                    View.MeasureSpec.UNSPECIFIED),  
                    View.MeasureSpec.makeMeasureSpec(i, View.MeasureSpec.UNSPECIFIED), mMeasuredDimension);  
              
            if (getOrientation() == HORIZONTAL) {  
                if (i == 0) {  
                    height = mMeasuredDimension[1];  
                }  
            } else {  
                if (i % span == 0) {  
                    height = (int) (height + mMeasuredDimension[1]);  
                }  
            }  
        }  
  
        switch (heightMode) {  
        case View.MeasureSpec.EXACTLY:  
            height = heightSize;  
        case View.MeasureSpec.AT_MOST:  
        case View.MeasureSpec.UNSPECIFIED:  
        }  
        setMeasuredDimension(widthSize, height);  
    }  
  
    private void measureScrapChild(RecyclerView.Recycler recycler, int position, int widthSpec, int heightSpec, int[] measuredDimension) {  
        if (position < getItemCount()) {  
            try {  
                View view = recycler.getViewForPosition(position);// fix // 动态添加时报IndexOutOfBoundsException  
                if (view != null) {  
                    RecyclerView.LayoutParams p = (RecyclerView.LayoutParams) view.getLayoutParams();  
                    int childWidthSpec = ViewGroup.getChildMeasureSpec(widthSpec, getPaddingLeft() + getPaddingRight(), p.width);  
                    int childHeightSpec = ViewGroup.getChildMeasureSpec(heightSpec, getPaddingTop() + getPaddingBottom(), p.height);  
                    view.measure(childWidthSpec, childHeightSpec);  
                    measuredDimension[0] = view.getMeasuredWidth() + p.leftMargin + p.rightMargin;  
                    measuredDimension[1] = view.getMeasuredHeight() + p.bottomMargin + p.topMargin;  
                    recycler.recycleView(view);  
                }  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
        }  
    }  
}  
```
通过LayoutManager去测量条目的宽高然后设置出高度.但是这种方法我在4.1的手机上运行正常,但是在7.0手机上依然显示不全(具体不知道哪个版本以上就不能完全显示);解决方法就是在RecyclerView外面包裹一层RealativeLayout;下面是代码修改前后的对比.
修改之前,4.1能显示完整,而7.0不能显示完整:

```

<ScrollView  
        android:layout_width="match_parent"  
        android:layout_height="match_parent">  
   <LinearLayout  
                android:layout_width="match_parent"  
                android:layout_height="match_parent"  
                android:orientation="vertical">  
                <TextView  
                    android:layout_width="match_parent"  
                    android:layout_height="100dp"  
                    android:text="文本"/>  
                <android.support.v7.widget.RecyclerView  
                    android:id="@+id/rv"  
                    android:layout_width="wrap_content"  
                    android:layout_height="0dp"  
                    android:layout_weight="1">  
                </android.support.v7.widget.RecyclerView>  
            </LinearLayout>  
</ScrollView>  

修改之后完整显示:

**[java]** [view plain](http://blog.csdn.net/kinly1993/article/details/54616348#) [copy](http://blog.csdn.net/kinly1993/article/details/54616348#)

<ScrollView  
        android:layout_width="match_parent"  
        android:layout_height="match_parent">  
        <RelativeLayout  
            android:layout_width="match_parent"  
            android:layout_height="wrap_content">  
            <LinearLayout  
                android:layout_width="match_parent"  
                android:layout_height="match_parent"  
                android:orientation="vertical">  
                <TextView  
                    android:layout_width="match_parent"  
                    android:layout_height="100dp"  
                    android:text="我是文本"/>  
                <android.support.v7.widget.RecyclerView  
                    android:id="@+id/rv"  
                    android:layout_width="wrap_content"  
                    android:layout_height="0dp"  
                    android:layout_weight="1">  
                </android.support.v7.widget.RecyclerView>  
            </LinearLayout>  
        </RelativeLayout>  
</ScrollView>  
```
3.手动设置LayoutParams,适用于题目上的任何一种情况
ListView和RecyclerView都能去获取item,然后测量出高度进行累加得到总高度再设置给自己,如果每条的高度已知并且高度都一致,那可以直接计算(注意加上分割线高度);如果是GridView或者多列RecyclerView.需要除以每行的条目数
listview:

```
LinearLayout.LayoutParams params = (LayoutParams) listview.getLayoutParams();  
int totalHeight = 0;  
for (int i = 0; i < list.size(); i++) {//list为数据集合,也可以通过adapter.getCount()得到  
    View listItem = buyAdapter.getView(i, null, listview);  
    listItem.measure(0, 0);  
    int perHeight = listItem.getMeasuredHeight();  
    totalHeight += perHeight;  
}  
float dividerHeight = listview.getDividerHeight();  
params.height = (int) ((list.size()-1)*dividerHeight+totalHeight);  
listview.setLayoutParams(params);  
```
RecyclerView:
```
**[java]** [view plain](http://blog.csdn.net/kinly1993/article/details/54616348#) [copy](http://blog.csdn.net/kinly1993/article/details/54616348#)

int h = 0;  
for (int i = 0; i < list.size(); i++) {  
    View view = recyclerView.getLayoutManager().getChildAt(0);  
    view.measure(0, 0);  
    h += view.getMeasuredHeight();  
}  
ViewGroup.LayoutParams layoutParams = recyclerView.getLayoutParams();  
layoutParams.height = h;  
recyclerView.setLayoutParams(layoutParams);  
```
注意:当视图未初始化完成时直接设置会报空指针,比如在onCreate方法中去设置,因为itemView还没有完全初始化,这时候可以通过设置view渲染完成的监听实现,以RecyclerView为例:

```
rootview.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {//rootView为根ViewGroup  
    @Override  
    public void onGlobalLayout() {  
        rootview.getViewTreeObserver().removeGlobalOnLayoutListener(this);  
        int h = 0;  
        for (int i = 0; i < list.size(); i++) {  
            View view = rv.getLayoutManager().getChildAt(0);  
            view.measure(0, 0);  
            h += view.getMeasuredHeight();  
        }  
        ViewGroup.LayoutParams layoutParams = rv.getLayoutParams();  
        layoutParams.height = h;  
        rv.setLayoutParams(layoutParams);  
    }  
});   
```