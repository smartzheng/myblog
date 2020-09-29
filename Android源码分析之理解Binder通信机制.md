+++
title= "Android源码分析之理解Binder通信机制"
date= "2019-10-23"
categories= [ "Android" ]
+++
**IPC，即Inner-Process-communication，进程间通信，是Android系统中比较难以理解的一个概念。而Binder则是Android系统中特有的进行IPC的一种方式，相对于Linux自带的其他方式（如管道）、socket、文件等而言，Binder具有更大的效率和安全优势。而本文将从各个层面深入探究Binder的原理。**

**一、Linux基础**  
本文首先 介绍部分和Android的进程间通信有关的Linux基础知识。
1.进程隔离/虚拟地址空间：Linux系统为了避免进程之间互相干扰，给各个进程分配了一块虚拟的地址空间，各个进程分别运行在各自的虚拟地址空间内。
2.系统调用：Linux提供的一种调用机制，通过系统调用，用户控件可以访问内核的应用程序。

**二、何为Binder**  
1.广义来讲，Binder可以理解为Android的一种跨进程通信方式。
2.从Android源码来看，Binder是一个Java类，实现了IBinder接口。
3.从Android的FrameWork来看，Binder可以看做是ServiceManager连接各种系统自带的Manager（如ActivityManager、WindowManager）和对应的Service（ActivityManagerService，WindowManagerService）的桥梁。
4.从应用层面来讲，Binder是客户端和服务端通信的媒介。当客户端对服务端进行绑定时，服务端会返回一个包含了服务端业务调用的Binder类对象，客户端可以通过此对象调用服务端的方法和数据。

**读到此处，相信大家还是很难对Binder的应用场景，实现方式和作用有很直观的理解，以下笔者将通过client调用**Service的详细过程**对其进行解析。**
****
**三、详解IPC**

![Binder](https://raw.githubusercontent.com/smartzheng/images/master/blog/Binder.webp)

1、我们将主要围绕上图进行分析。  
1）图中有四个关键的对象：Server、Client、SeviceManager（图中最下方的矩形）、Binder驱动。其中前三者运行在客户空间，而Binder驱动运行在内核空间。Server、Client由应用程序负责提供，而Binder驱动和SeviceManager由系统提供。  
2）Binder进程间通信实在OpenBinder的基础上实现的，其中提供服务的进程分为服务进程，而访问服务的进程分为Client进程，两者分别运行在不同的进程中。而Server进程和Client进程需要通信，需要借助运行在内核空间的Binder驱动，Binder驱动对用户空间暴露了一个设备文件、dev/binder，应用程序可以依靠他来建立通信通道。  
3）当Server空间中的Service启动时（不一定是Service子类，但需要继承IInterface接口），会在ServiceManager（下称SM）中去进行注册。SM可以看做是Binder进程通信间的上下文管理者，他也需要与Server和Client通信，可以看做是特殊的Service组件。  
4）当Client需要调用Service（再次强调，这里指的Service指的是继承了IInterface的一个实现类，通常还有一个中间接口IService）中的方法或者数据时，需要根据类名在SM中进行查找，SM会返回服务端的代理对象，代理对象（Proxy）实现了Service所对应的接口（假设命名为IService，继承关系为：Service>IService>IInterface）的子类Stub，并对IService中定义的方法做了封装，使其可以实现进程间读写数据（需要需要序列化），因此代理对象可以将数据交由IService的最终实现类进行处理，完成之后再返回给Client。    
**上面就是Binder进程间通信机制的大致流程。下面将在分别对SM，Service组件启动和其代理对象具体获取过程作详细介绍。**  
5）SM：SM由系统负责启动，启动过程分为三个步骤：打开设备文件dev/binder将其映射到本进程的地址空间；将自己注册为Binder进程间通信的额上下文管理者；调用函数binder_loop来循环等待和处理Client的通信请求。SM中含有四个方法getService、checkService、addService、listService，Client通过getService获取Service的代理对象，Service通过addService将自己注册进SM中。SM运行在自己的独立空间，也继承了IInterface，所以它自身和Server及Client通信也需要跨进程：当Client需要通过SM的getService方法获取Service代理对象或者Service需要通过addService进行注册时时，也需要先获取SM的代理对象，但是不同的是其代理对象获取比较简单，Android系统在Binder库中提供了一个函数defaultServiceManager，通过此方法即可获取到。  
6）Service启动过程：Service组件是在Server进程中运行的，Server进程启动时，首先将它里面的Service组件注册到SM中，接着再启动一个Binder线程池来等待和处理Client进程的通信请求。  
7）Service代理对象获取：参考AIDL的解析。
