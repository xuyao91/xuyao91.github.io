---
layout: post
title: "App自动化测试平台搭建记录之什么是appium"
date: 2016-07-09 17:06:02 +0800
comments: true
categories: 
---
####前言
目前我们的项目关于测试这块还是处于很初级阶段，rails后端的rspec也写的很少，app方面完全靠手动测试，基于此情况，我们决定搭建一个自动化测试平台，期望目标是移动端ios和android及h5每天定时拉取最新代码，进行部署，打包，最后进行回归测试，生成测试报告发送给测试人员，
最后流程如下所示
<!-- more -->
![img](http://upload-images.jianshu.io/upload_images/1796624-10f320a9d37a0d96.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####为什么选择appium作为自动化测试框架
*   使用标准的api形式，能够同时支持ios，android，h5,接口保持统一，减少开发维护成本
*   你可以使用任何你喜欢的开发工具，及支持多种脚本语言(ruby/python/java/php/javascript/C#/ Clojure/Perl)  
*   你可以使用任何的测试框架
####What is the Appium
具体什么是appium呢，引用了官方的一段话：
  Appium is an open source test automation framework for use with native, [hybrid](http://appium.io/slate/en/master/#hybrid.md) and mobile web apps.

####Appium Concepts（appium理念）  
**Client/Server 设计风格**   
   Appium本质是一个核心webserver,暴露一些REST API与手机设备交互, 而且之间通信是一标准的HTTP协议，client的test code语言几乎现在主流的语言都支持，写测试用例比较方便，下图是它servre端的GUI

![CDF57F68-BBAE-496C-B12D-DA3F69B8A77D.png](http://7xjibn.com1.z0.glb.clouddn.com/CDF57F68-BBAE-496C-B12D-DA3F69B8A77D.png)


**Session(会话)**  
  自动化每次都会在一个会话里面进行，client每次都会发送一个post /session 请求，服务器根据客户端信息起一个automation session和返回一个session ID用于后续操作

**Desired Capabilities**  
  一组hash参数，告诉server端我们需要启动一个的怎样automation session,不同设备参数不同
这是iOs一个demo
```ruby
capabilities = {
	'appium-version': '1.0',
	'platformName': 'iOS',
	'platformVersion': '8.2',
	'deviceName': 'iPhone 6',
	'app': "/Users/xuyao/Downloads/Cloudoc-Patient.app"
}
```
这是android的样本
```ruby
capabilities = {
	'appium-version': '1.0',
	'platformName': 'Android',
	'platformVersion': '5.1',
	'deviceName': 'test',
	'app': '/Users/xuyao/Downloads/Yydys0627.apk',
}
```

**Appium Server**  
  server端是用node.js写的

**Appium Clients**  
  客户端的库有很多( Java, Ruby, Python, PHP, JavaScript, and C#)，这些都对WebDriver封装扩展过,所以不需要找个WebDriver client来做这些东西,如果使用ruby语言来写脚本，ruby里有个[appium_lib](https://rubygems.org/gems/appium_lib) gem，里面集成了很多现成的方法，可以再安装一个[appium_console](https://github.com/appium/ruby_console)，直接打开console进行调试

####Appium Inspector
在控件台里面获取页面元素比较麻烦，往往需要一直定位，Appium提供了一个比较友好的GUI给我们操作，而且操作起来还容易，大大减少了我们查找元素的时间
![Appium Inspector GUI](http://7xjibn.com1.z0.glb.clouddn.com/1796624-36371e4f4fe1beb2.jpeg)

######参考资料
http://appium.io/  
https://github.com/appium/ruby_console  
https://github.com/appium/ruby_lib  
https://ruby-china.org/topics/30074
