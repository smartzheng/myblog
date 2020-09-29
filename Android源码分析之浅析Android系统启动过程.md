+++
title= "Android源码分析之浅析Android系统启动过程"
date= "2017-09-27"
categories= [ "Android" ]
+++
##### 0.Android的两个世界  
*  Android系统存在两个世界：Java世界和Native（C或者C++的世界）世界，而大部分应用层开发者除了少数时候用到JNI之外，接触到的通常只是Java世界。Android系统基于Linux内核，最早当然是Native世界。在启动电源将ROM加载到RAM之后，BootLoader启动系统，进入Android内核层。内核启动时，初始化进程管理、内存管理，加载Display,Camera Driver，Binder Driver等相关工作，并创建守护进程。至此即完成底层初始化，进入Native层。

先放一张自己画的流程图

![Android应用启动](https://raw.githubusercontent.com/smartzheng/images/master/blog/Android应用启动.webp)

##### 1.init进程    
*  init进程是Linux系统**用户空间**的第一个进程，在系统启动时会首先被启动，其对应的文件为init.c。  
*  init进程的主要任务就是解析配置文件，启动zygote进程及属性服务（propertyservice），（属性服务类似于Windows系统的注册表，存储一些key/value的键值对。应用程序可以通过这个属性机制设置属性。当系统重启或应用重启时从中读取数据进行恢复，一般运用于系统应用。此处不作过多讨论）。并且在此之后进入一个死循环，等待处理来自socket和属性服务器的事件。我们重点在zygote的启动。
*  在init.c的service_start()方法中，调用fork()函数创建一个子线程（fork是Linux创建线程的一种方式），并在此线程中进入到app_process的main函数中。实际上zygote的原名就叫做app_service,只不过在调用的过程中被改名为zygote（受精卵），原型app_service对应的源文件是App_main.cpp。  

##### 2.App_main.cpp  
-   App_main.cpp文件的入口为main()，在主方法中调用runtime.start()；runtime的类型为APPRuntime，所以zygote的主要功能交给了APPRuntime处理。

##### 3.AppRuntime  
-  AppRuntime由AndroidRuntime派生而来，在start()方法中主要做了三件事情：startVm()，startReg()和CallStaticVoidMethod，即创建虚拟机,注册JNI函数,再通过JNI调用  ZygoteInit.java的main()函数，由此开创了Android的Java世界。  

##### 4.zygote  
-  在ZygoteInit类的main()内部，进行了4个重要操作：
>  ①建立IPC通信服务端：registerZygoteSocket; zygote及其他系统程序的通信并未使用Binder，而是使用Socket，此方法即是建立这个Socket的服务端。 
>  ②预加载类和资源：preloadClasses和preloadResources；通过反射原理对系统所用的类和资源进行预加载，缺点是导致了Android系统启动较慢。  
>  ③启动system_server进程：startSystemServer()； system_server是Java世界的核心，即framework的核心。如果system_server死亡，则zygote将会自杀。 startSystemServer()最终也是通过zygote的分裂（fork）而来（最终实现是在native层）。  
>  ④等待处理请求：runSelectLoopMode()；用于处理客户连接和客户请求。  

-  在处理完以上的主要任务后，zygote将暂时退出舞台等待被唤醒执行任务（fork下一个进程），而后面的任务则交给了他分裂的重要进程system_server进行执行； 

##### 5.system_server  
-  system_server(下简称SS)的产生是在native层（dalvik_system_Zygote.c中），在创建之后调用handleSystemServerProcess完成自己的使命。期间会先与Binder通信系统建立联系，这样SS就能使用Binder。辗转之后system_server最终调用了Java层的com.android.sever.SystemServer类的main函数。  
-  在SystemServer类的main函数中，分别调用了native层的init1()和Java层的init2()。init1()创建了一些系统服务，并且把调用线程加入到Binder通信中；init2()启动了ServerThread线程，启动系统的各项服务，如PowerManager,WatchDog,WindowManager，ActivityManager，可谓十分重要。

##### 至此，Android启动的大致流程介绍完成，前面提到，zygote在分裂出SS之后，会runSelectLoopMode来等待处理客户端的消息，比如创建新进程或者说启动新的应用的请求。下篇文章再对此进行分析。