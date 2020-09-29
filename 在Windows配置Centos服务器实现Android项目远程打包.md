+++
title= "在Windows配置Centos服务器实现Android项目远程打包"
date= "2017-09-29"
categories= [ "Android" ]
+++
#### 0.需求
平时开发测试环节中涉及多次回归测试，于是考虑对服务器进行配置，当开发完后每次push代码，测试部门即可从服务器打包pull进行测试。
#### 1.连接服务器
我用的是PuTTY 进行连接，软件可自行Google下载，安装完成之后填写id和password即可登录到CentOs服务器。（本文CentOS的版本为6.8，且已配置好JDK）
#### 2.配置AndroidSDK
##### 1）下载sdktools
-  进入etc，创建文件夹  
```
cd /etc  
```  
```  
mkdir androidSdk
```  

-  下载，解压  
```
wget https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip  
```  
```
unzip sdk-tools-linux-3859397.zip 
 ```

##### 2）配置环境变量  
-  打开etc目录下的profile  
```
cd etc
```  
```  
vi profile
```
-  打开后输入i进入编辑模式，在文件末尾插入以下命令  
```
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL 
 ```  
```
export PATH=$PATH:/etc/androidSdk/tools/bin
```
-  esc退出编辑，:w进行保存，:q退出文件
-  执行：```source profile  ``` 生效
-  执行```sdkmanager```检查是否安装成功
##### 3）安装必要的package
-  根据需要执行安装以下的package
build-tools;25.0.3                
emulator                   
extras;android;m2repository
patcher;v4                 
platform-tools                    
platforms;android-25            
tools    
安装命令为：```sdkmanager "build-tools;26.0.0"```  
将命令中的build-tools;26.0.0依次替换为上面的包名称+版本号可依次安装，也可选择需要的版本。  
##### 4）配置platform tools
-  在etc/profile中添加以下命令，方法同上面配置tools  
```export PATH=$PATH:/etc/androidSdk/platform-tools```      
-  配置完成后可以执行adb命令检查
#### 3.配置gradle
##### 1）下载         
-  进入etc，创建文件夹  
```
cd /etc  
```  
```  
mkdir gradle
```  

-  下载，解压  
```
wget https://services.gradle.org/distributions/gradle-4.0.1-bin.zip  
```  
```
unzip gradle-4.0.1-bin.zip 
 ```
##### 2）配置gradle
-  在etc/profile中添加以下命令，方法同上面配置tools  
```
export PATH=$PATH:/etc/gradle/gradle-4.0.1/bin
```      
-  配置完成后可以执行gradle命令检查
#### 4.配置git
-  直接执行  
```yum install git```  
#### 5.完成  
-  接下来可以直接pull代码，进入到项目根目录执行```gradle assRelease```打包。