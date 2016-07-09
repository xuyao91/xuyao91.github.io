---
layout: post
title: "App自动化测试平台搭建记录之Android模拟器/真机的配置"
date: 2016-07-10 00:12:30 +0800
comments: true
categories: 
---
上一篇讲了安装appium及ios的配置，如果没看过这篇文章，建议先看一下，[传送门](http://www.jianshu.com/p/8ac5491436b2)
既然是配置Android模拟器，那么Java和Android sdk的安装是必不可少的了
<!-- more -->
####安装Java环境  
安装Java环境就不多讲了，大家各显神通吧
####安装Android SDK  
到Android[官网](https://developer.android.com/studio/index.html)，下载一个mac版的sdk,解压出来
进入tools文件夹，执行android文件，出现如下界面，选择需要安装的版本
![android_sdk_download](http://7xjibn.com1.z0.glb.clouddn.com/android_sdk_download.png)
下载完成后,进入android sdk目录下面执行
```ruby
android avd
```
打开android avd管理界面，添加一个avd设备
![avd_manger](http://7xjibn.com1.z0.glb.clouddn.com/avd_manger.png)
![create_avd](http://7xjibn.com1.z0.glb.clouddn.com/create_avd.png)
新建完成后，可以启动看一下android 虚拟设备  
最后进入platform-tools使用命令，查看当前的android可用设备
如果是真机测试的话，使用USB线连接电脑，打开开发者模式，同意此电脑连接设备，appium启动时,会查找当前可用的android设备
```ruby
adb devices
#这里会显示当前可用的android设备
List of devices attached
88EKBME225K8	device
```
最后配置环境变量，环境变量包括JAVA_HOME和ANDROID_HOME
编辑.profile（如果安装了zshrc，应该是.zshrc）
添加以下代码
```ruby
export ANDROID_HOME=/Users/xuyao/Downloads/android-sdk-macosx
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_71.jdk/Contents/Home
PATH=$PATH:/usr/local/share/npm/bin/
PATH=$PATH:$ANDROID_HOME/build-tools
PATH=$PATH:$ANDROID_HOME/platform-tools
PATH=$PATH:$ANDROID_HOME/tools
export PATH
```   
保存后，为使其生效，需要重新sources一下,source .profile(.zshrc)  
输出一下，检查是否成功
```ruby
echo $ANDROID_HOME
/Users/xuyao/Downloads/android-sdk-macosx
echo $JAVA_HOME
/Library/Java/JavaVirtualMachines/jdk1.8.0_71.jdk/Contents/Home
```
一切准备就绪，接下来就开始在appium里配置android信息了
![android_config](http://7xjibn.com1.z0.glb.clouddn.com/android_config.png)
platform Versiont选择的时候，如果是真机测试时，确定所选版本和机子上的android版本一样，否则会报错。
配置完成后，接下来就是启动server和client,这里的步骤一ios是一样的，具体可查看上一篇文章，截一个android版本的client启动成功的Inspector gui的图
![img](http://7xjibn.com1.z0.glb.clouddn.com/android_inspect.png)

######注意的地方
如果启动client的时候报下面的错
```ruby
[MJSONWP] Encountered internal error running command: Error: Could not find a connected Android device.
```
说明client端找不到可以连接的android设备，检查一下avd是否启动或者真机连接是否正确  
######参考资料
https://developer.android.com/studio/index.html

