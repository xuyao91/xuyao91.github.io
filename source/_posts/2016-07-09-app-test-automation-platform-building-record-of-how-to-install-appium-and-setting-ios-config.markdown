---
layout: post
title: "App自动化测试平台搭建记录之appium安装及ios模拟器/真机的配置"
date: 2016-07-09 21:29:56 +0800
comments: true
categories: 
---
上一篇讲了什么是appium，这次讲一下如何安装appium，及怎样配置ios/android的模拟器/真机
####系统选择  
因为要跑ios的包的，所以我们肯定需要apple的系统来支持，而且ios版的appium也支持android，这样就完美了。
<!-- more -->
######iOS必备条件
*   Mac OS X 10.10 or 更高, 建议使用10.11.1 
*   XCode >= 6.0, 7.1.1 
*   Apple Developer Tools (iPhone simulator SDK, command line tools)  

######Android必备条件
*   [Android SDK](http://developer.android.com/) API >= 17 (Additional features require 18/19)  

####安装appium  
appium服务器是使用node.js跑的,所以要确保你机子上安装了node.js
```ruby
brew install node #安装nodejs
npm install -g appium #安装appium
appium  #启动appium
```
当然你还可以下载一个包，直接安装，[点击下载](https://bitbucket.org/appium/appium.app/downloads/)
如果不确实appium是否安装好，可以使用appium-doctor插件检测依赖环境是否安装成功
```ruby
npm install -g appium-doctor # 安装appium-doctor
appium-doctor #如果安装成功，会显示以下数据
info AppiumDoctor ### Diagnostic starting ###
info AppiumDoctor  ✔ Xcode is installed at: /Applications/Xcode.app/Contents/Developer
info AppiumDoctor  ✔ Xcode Command Line Tools are installed.
info AppiumDoctor  ✔ DevToolsSecurity is enabled.
info AppiumDoctor  ✔ The Authorization DB is set up properly.
info AppiumDoctor  ✔ The Node.js binary was found at: /usr/local/bin/node
info AppiumDoctor  ✔ HOME is set to: /Users/xuyao
info AppiumDoctor  ✔ ANDROID_HOME is set to: /Users/xuyao/Downloads/android-sdk-macosx
info AppiumDoctor  ✔ JAVA_HOME is set to: /Library/Java/JavaVirtualMachines/jdk1.8.0_71.jdk/Contents/Home
info AppiumDoctor  ✔ adb exists at: /Users/xuyao/Downloads/android-sdk-macosx/platform-tools/adb
info AppiumDoctor  ✔ android exists at: /Users/xuyao/Downloads/android-sdk-macosx/tools/android
info AppiumDoctor  ✔ emulator exists at: /Users/xuyao/Downloads/android-sdk-macosx/tools/emulator
info AppiumDoctor ### Diagnostic completed, no fix needed. ###
info AppiumDoctor 
info AppiumDoctor Everything looks good, bye!
info AppiumDoctor 
```
好了以上就是安装appium的步骤，安装好，就可以启动了，启动后的界面如下
![CDF57F68-BBAE-496C-B12D-DA3F69B8A77D.png](http://7xjibn.com1.z0.glb.clouddn.com/CDF57F68-BBAE-496C-B12D-DA3F69B8A77D.png)  
####iOS模拟器/真机信息配置
正常情况下apple的包是.ipa文件，那里因为所有的ios设备使用的都的ARM处理器，但是我们现在配置的是模拟器的设备，而模拟器是基于Intel处理器,处理架构不一样，所以在编辑生成包的时候请选择.app文件，请看下图的配置参数
![apple-setting](http://7xjibn.com1.z0.glb.clouddn.com/apple-setting.png)
App path就是apple包的路径  
BunleID就是苹果开发的bundle identifier，仅在真机测试时勾上并填写  
UDID苹果设备的唯一ID，仅在真机测试时勾上并填写  
Force Device 要测试的iphone设备
Platform Version 要测试的iphone的版本好(设备和版本号一定要匹配)
以上配置好后，点击launch按钮，启动appium服务，启动成功后，可看到如下信息
```ruby
[MJSONWP] Responding to client with driver.getStatus() result: {"build":{"version":"1.5.3"...
[HTTP] <-- GET /wd/hub/status 200 26 ms - 83 
```
接下来点击那个类似搜索的按钮来启动client端  
如果启动的时候报如下错误
```ruby
[MJSONWP] Encountered internal error running command: Error: Could not find a device to launch. You requested 'iPhone 6 (9.3 Simulator)',   
but the available devices were: ["Resizable iPad (8.2 Simulator) [341C7D49-B181-46B8-AB61-CF644C3250F1]","Resizable iPhone (8.2 Simulator) [411D39CB-6DB6-41A9-9E6C-89CFDED27B69]","iPad 2 (8.2 Simulator) [B3780274-2AE7-4677-831A-17CDDC92F2A5]","iPad Air (8.2 Simulator) [C675C17C-9B79-4E57-B877-732A441F8DDA]","iPad Retina (8.2 Simulator) [1C9FF2EF-A643-4D27-9268-4C3A4ADD8710]","iPhone 4s (8.2 Simulator) [0AB005A2-63AD-44F8-AE80-8BFBB21BCDD2]","iPhone 5 (8.2 Simulator) [15765CF4-96B1-4826-B397-6E23658AC89A]","iPhone 5s (8.2 Simulator) 
```
那是因为你的设备和版本不对的原因，仔细看下log就发现了，改一下设备和版本信息重新启动  
启动成功后出现如下画面，启动成功后会帮你打开Appium Inspector，可以使用Inspector查找组件路径，自此苹果的设备就配置成功了
![iphone5](http://7xjibn.com1.z0.glb.clouddn.com/iphone_5s_device.png)
![Appium Inspector GUI](http://7xjibn.com1.z0.glb.clouddn.com/1796624-36371e4f4fe1beb2.jpeg)
#####参考资料
http://appium.io/getting-started.html?lang=zh
https://ruby-china.org/topics/30085

