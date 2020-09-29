+++
title= "Navigation返回Fragment重走onCreateView问题"
date= "2019-05-31"
categories= [ "Android" ]
+++

>在使用Navigation的过程中，发现其页面跳转效率确实很不错，XML管理页面跳转逻辑以及fragment之间的参数传递使用起来都很方便，但是一个很大的问题就是在fragment出栈返回上一页时，上一个fragment会重走onCreateView方法。而我们的很多view和数据初始化工作都是在onViewCreated之后进行的，导致的结果是每次回上一个页面可能会重新刷新，这一点体验很差。这里提供一个方法来避免每次重新创建view。

编写一个BaseNavigationFragment:

```
class BaseNavigationFragment : BaseFragment() {
    protected var isNavigationViewInit = false//记录是否已经初始化过一次视图
    private var lastView: View? = null//记录上次创建的view
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        //如果fragment的view已经创建则不再重新创建
        if (lastView == null) {
            lastView = super.onCreateView(inflater, container, savedInstanceState)
        }
        return lastView
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        if(!isNavigationViewInit){//初始化过视图则不再进行view和data初始化
            super.onViewCreated(view, savedInstanceState)
            initView(view)
            initData()
            isNavigationViewInit = true
        }
    }
}
```

核心思路在上面的注释，即保存上次创建的view，返回上一页再次走onCreateView时直接将其返回，并且在onViewCreated方法中不再进行初始化工作。  
leak canary可能会提示lastView内存泄漏，忽略即可。