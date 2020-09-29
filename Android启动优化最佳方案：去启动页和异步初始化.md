+++
title= "Android启动优化最佳方案：去启动页和异步初始化"
date= "2019-04-25"
categories= [ "Android" ]
+++
项目地址：[https://github.com/smartzheng/asyncstarter](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsmartzheng%2Fasyncstarter)

随着APP的日渐增大，集成的三方库也越来越多，导致APP的启动极其缓慢。最近在慕课get了一些不错的优化方案，将原来的冷启动时间大概提升30%。
启动的时间监测可以直接用adb命令实现：
>adb shell am start -W PackageName/ActivityName  
  
下面是我未优化之前的项目debug版本启动时间（华为p10plus），这里介绍一下几个概念  
ThisTime：最后一个Activity启动耗时
TotalTime：所有Activity启动耗时
WaitTimeTime：AMS启动Activity启动耗时  
可以看到耗时接近1.3s多（1.3s不算长，但是往往应用加固之后还会慢大一截）。
```
-> ~ adb shell am start -W com.smartzheng/com.smartzheng.activity.SplashActivity
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.
LAUNCHER]cmp=com.smartzheng/.activity.MainActivity }
Warning: Activity not started, its current task has been brought to the front
Status: ok
Activity: com.uoko.mlgb/.mvp.view.activity.MainActivity
ThisTime: 600
TotalTime: 1301
WaitTime: 1330
Complete
```

优化一：去掉启动页
IPC是个比较耗时的操作，往往我们会设置一个闪屏页，去掉之后可以一定幅度减少启动时间。  
我的做法是直接删除SplashActivity，将MainActivity设为启动页。然后在manifests中将其theme设为启动时的theme：  
```xml
<activity
    android:name=".activity.MainActivity"
    android:configChanges="screenSize|keyboardHidden|orientation"
    android:launchMode="singleTask"
    android:screenOrientation="portrait"
    android:theme="@style/SplashTheme">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
    </intent-filter>
    <intent-filter>
        <data android:scheme="growing.a4ce2f9edf74350a"/>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
    </intent-filter>
</activity>
```
SplashTheme中windowBackground为启动图片（提一句：这里的windowBackground最好是9patch图片，防止拉伸变形），这样设置还有一个好处是可以解决掉APP启动白屏的问题。  

```xml
<style name="SplashTheme" parent="AppThemeFullScreen">
    <item name="android:windowBackground">@drawable/splash</item>
    <item name="android:windowTranslucentNavigation">true</item>
</style>
```

最后，在MainActivity的onCreate方法中，手动setTheme更改为Application的默认Theme：  

```java
override fun onCreate(savedInstanceState: Bundle?) {
    setTheme(R.style.AppTheme)
    super.onCreate(savedInstanceState)
}
```

评论中有人问到如果去掉闪屏页，那广告位怎么解决？其实广告页面的显示其实用一个单独的Activity实现不是一个好选择，直接调用Activity的addContentView方法就可以添加一个view在activity上面，在倒计时完成之后再removeContentView，这样随时都可以在任意activity上添加广告，通用性和性能都更好。  

优化二：异步初始化  
优化之前，我的Application的onCreate是这样的： 

```java
override fun onCreate() {
    super.onCreate()
    installSysExceptionHandler()
    initImPush()
    initGrowing()
    initBaidu()
    initApollo()
    initBugly()
    initIMSdk()
    initPush()
    initRxJava()
    initNineGridView()
    initSeoReport()
    initStrictMode()
    initFonts()
}
```

每一项初始化任务都是同步执行的。优化的思想就是将各个初始化任务放到一个线程池中去执行。优化之后代码如下：  
```java
override fun onCreate() {
    super.onCreate()
    TaskDispatcher.init(this)
    TaskDispatcher.createInstance().run {
        addTask(InitApolloTask())
            .addTask(InitBaiduTask())
            .addTask(InitBuglyTask())
            .addTask(InitFontTask())
            .addTask(InitGrowingTask())
            .addTask(InitIMPushTask())
            .addTask(InitIMSDKTask())
            .addTask(InitJPushTask())
            .addTask(InitViewTask())
            .addTask(InitRxjavaTask())
       start()
       await()
    }
}
```
看看优化之后的启动耗时（release环境下），启动速度大概提升了近30%：

```
-> ~ adb shell am start -W com.smartzheng/com.smartzheng.activity.SplashActivity
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.
LAUNCHER]cmp=com.smartzheng/.activity.MainActivity }
Status: ok
Activity: com.uoko.mlgb/.mvp.view.activity.MainActivity
ThisTime: 911
TotalTime: 911
WaitTime: 931
Complete

```
将各个初始化任务放到线程池执行需要解决很多问题：  
部分初始化工作只能在主线程执行  
部分初始化任务需要先执行  
部分初始化任务需要依赖其他任务执行完之后才可以执行  
部分任务必须执行完才能进入Activity页面
参考课程，将封装的代码放到了[GitHub](https://github.com/smartzheng/asyncstarter)，可以直接依赖使用；具体的实现细节在源码中有详细注释。使用方法如下： 
 
1.添加仓库

```
allprojects {
	repositories {
		...
		maven { url 'https://jitpack.io' }
	}
}
```

2.添加依赖  

```
dependencies {
    implementation 'com.github.smartzheng:asyncstarter:1.0.1' 
}
```
  
3.自定义Task

```java
    class InitTask : Task() {
        override fun needWait(): Boolean {//是否需要在阻塞在await(),在Application的onCreate方法之前执行完
        return true
        }

        override fun dependsOn(): MutableList<Class<out Task>> {//等待另一个Task执行完再执行此任务初始化
            return mutableListOf(InitTask1::class.java)
        }
        override fun runOnMainThread(): Boolean {//是否需要运行在主线程
            return true
        }
        override fun needRunAsSoon(): Boolean {//提高优先级,也可以指定优先级大小priority
            return true
        }
        override fun run() {
            //初始化
        }
    }
```

4.在Application中

```java
    class MyApplication : Application() {
        override fun onCreate() {
            super.onCreate()
            AsyncStarter.init(this)
            val starter = AsyncStarter.createInstance()

            starter.addTask(InitTask1())
                .addTask(InitTask2())
                .addTask(InitTask3())
                //addTask()...
            starter.start()
            starter.await()
            //Kotlin run
            //AsyncStarter.createInstance()
            //            .run {
            //                addTask(InitTask1())
            //                    .addTask(InitTask2())
            //                    .addTask(InitTask3())
            //                    //addTask()...
            //                    .start()
            //                await()
            //            }
        }
    }
```  

详细的配置解释和实例可以查看GitHub源码，其中有详细注释，欢迎star：[https://github.com/smartzheng/asyncstarter](https://github.com/smartzheng/asyncstarter)










