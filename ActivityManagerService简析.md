+++
title= "ActivityManagerService简析"
date= "2019-03-05"
categories= [ "Android" ]
+++
#### AMS在Android系统中扮演很重要的角色，主要负责系统中四大组件的启动、切换、调度及应用进程的管理和调度等工作，其职责与操作系统中的进程管理和调度模块相类似。当发起进程启动或者组件启动时，都会通过Binder通信机制将请求传递给AMS，AMS再做统一处理。以下以启动Activity为例简析AMS的管理工作。  
  
##### AMS在SystemServer进程中启动，SystemServer的main方法会调用startBootstrapServices方法启动AMS
##### 7.0及之前，AMS通过代理模式来完成Binder通信：
Activity的直接管理者是ActivityManager，但最终管理者是AMS：当Client端发起启动Activity请求后，AM会通过ActivityManagerNative的getDefault来得到其内部类ActivityManagerProxy的单例对象，即AMS在客户端（用户进程）的代理对象，作为代理类，AMP中含有AMS的引用，AMN和AMP都实现了IActivityManager，IActivityManager继承了IInterface（实现Binder通信的必备条件），所以AMP具备了Binder通信能力，statActivity最终会通过AMP中的AMS引用来调用AMS的transact方法，向AMS发送启动Activity请求，并将序列化数据传递给AMS，随后AMS的子类AMN的onTransact会执行，它会将具体的启动工作交给ActivityStater来负责。  具体流程及关系图如下：

![AMS.png](https://raw.githubusercontent.com/smartzheng/images/master/blog/AMS.webp)

##### 8.0之后，AMS通过AIDL完成Binder通信。具体实现比较简单。

##### ActivityRecord、TaskRecord和ActivityStack
AMS中主要涉及这三个数据结构：
ActivityRecord：存储Activity的相关信息，比如AndroidMainifes的节点信息，启动Activity的包名，所在进程，图标主题标识符，当前Activity状态，所属TaskRecord等。
TaskRecord：描述一个Activity任务栈，主要维护了一个按历史顺序排列的ArrayList<ActivityRecord>，并包含此任务栈所属的ActivityStack等。
ActivityStack：一个管理系统中所有Activity的管理类，真实交由ActivityStackSupervisor管理，内部维护了Activity的所有状态，并对不同状态的Activity进行分类管理，如最近启动的Activity，正在暂停的Activity等。
##### 可以借助下图理解三者关系：

![ActivityRecord、TaskRecord、ActivityStack.png](https://raw.githubusercontent.com/smartzheng/images/master/blog/ActivityRecord、TaskRecord、ActivityStack.webp)
