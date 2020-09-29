+++
title= "SwipeRefreshLayout手动调用setRefresh(true)不起作用的原因"
date= "2017-09-22"
categories= [ "Android" ]
+++

1.现象
往往发生这个现象是出现在onCreate中去调用这个方法,会发现不起作用.原因是在onCreate方法中view未加载完全,所以不能显示.

2.解决  
方法一:用post队列 
 
```java 
refreshLayout.post(new Runnable() {  

     @Override    
     public void run() {  
        refreshLayout.setRefreshing(true);    
        onRefresh();//是否触发onRefresh中的方法    
     }    
});    
```  
 方法二:手动设置加载图形的偏移量(相对于手动去初始化对应的view)

```
refresh.setProgressViewOffset(false, 50, 100);  //依次代表:是否缩放加载圆圈,偏移量顶部y值,偏移量底部y值  
refresh.setRefreshing(true);      
```

方法三:手动监听SwipeRefreshLayout的measure方法完成后再去setRefreshing(true)

```
rootView.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {  
    @Override  
    public void onGlobalLayout() {  
        root.getViewTreeObserver().removeGlobalOnLayoutListener(this);  
        mRefreshLayout.setRefreshing(true);  
    }  
});   
```