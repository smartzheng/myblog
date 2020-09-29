+++
title= "Android进程启动流程简析"
date= "2019-02-22"
categories= [ "Android" ]
+++
自己画了一张简单流程图：  

![Android进程启动流程](https://raw.githubusercontent.com/smartzheng/images/master/blog/Android进程启动.jpeg)

启动一个新进程主要涉及四步：
1、发送启动进程的需求，可能是Launcher启动一个新的APP，或者APP开启独立进程或其他APP  

2、发送需求的进程通过Binder通信机制与SystemServer进程通信，通过运行在其中的ActivityManagerService来执行下一步流程  

3、AMS通过Socket将创建新进程的请求发送给Zygote进程，在Zygote启动后会执行runSelectLoop方法来等待创建进程请求，当接收到AMS的请求之后fork出新进程，并创建Binder线程池，将新进程的主线程加入到其中，这样新进程也可以使用Binder通信机制了。

4、Zygote进程通过Binder与新开启的应用进程通信，ActivityThread的内部类ApplicationThread作为跨进程通信的桥梁（8.0之后采用AIDL实现，之前使用的是代理模式），ApplicationThread运行在Binder线程中，所以最后需要通过线程切换（H.sendMessage）来进入主线程

#### 由此可见，一个新进程的创建涉及4个进程，3次跨进程通信，2次为Binder，1次为Sockt