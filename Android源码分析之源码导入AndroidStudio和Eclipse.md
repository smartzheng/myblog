+++
title= "Android源码分析之源码导入AndroidStudio和Eclipse"
date= "2017-07-08"
categories= [ "Android" ]
+++
之前学习Android源码的时候用的sourceinsight，始终感觉类跳转不大精确而且使用久了会有卡顿，就一直想着如何能在Androidstudio或eclipse上查看在所有源码，网上查了很多资料大致有了思路。

一、使用AndroidStudio

第一步：安装VMware。这个很简单,官网下载安装(我用的云盘：[http://pan.baidu.com/s/1gfxen7t](http://pan.baidu.com/s/1gfxen7t))就行(如果慢的话可以在用百度网盘搜索：[http://www.sobaidupan.com/](http://www.sobaidupan.com/))。我下载的是12.5.7版本。

第二步：安装Ubuntu。安装过程也很简单，参考百度经验足矣：[http://jingyan.baidu.com/article/14bd256e0ca52ebb6d26129c.html](http://jingyan.baidu.com/article/14bd256e0ca52ebb6d26129c.html)。

第三步：安装VMware-tools。因为需要将windows中的文件复制到Linux，所以需要安装此插件(也可以直接在Linux上下载，不过速度很慢)，也可以起到桌面全屏的效果，不然一小块界面很影响操作，点击虚拟机>>安装VMware tools，系统会下载对应的文件VMware tools，并加入驱动。点击进入该目录(文件夹中会有vmware-install.pl文件) 右键打开命令行终端，执行sudo  ./vmware-install.pl，输入密码完成安装。安装完成后重启Ubuntu生效,就可以直接将windows文件拖入Linux系统中。若直接拖入已解压文件会很慢，所以选择拖入压缩包再进行解压，Linux默认不支持rar，所以若为rar文件需要安装三方解压软件(unrar或者ark,不过亲测更改后缀为Zip有效)，读者自行安装,此处不再赘述（最好在Linux上用repo和git直接下载）。

第四步：编译Android源码。具体参考[http://www.jianshu.com/p/367f0886e62b](http://www.jianshu.com/p/367f0886e62b)。因为最后我选择使用eclipse进行源码查看。

二、使用eclipse
使用eclipse就比较简单了，而且对电脑内存和硬盘的要求很低。

第一步、下载源码并解压，出现文件名重复直接覆盖。

第二步、将\development\ide\eclipse下的。classpath文件拷贝到源码根目录。

第三步、eclipse新建Java工程（不是AndroidProject，右键new--javproject），去掉勾选use degault location，选择为源码文件夹目录。点击finish完成。

第四步、源码已经导入完成，但是会发现很多报错，因为是Java工程的原因。建议将error下划线去掉，改善一下阅读体验。