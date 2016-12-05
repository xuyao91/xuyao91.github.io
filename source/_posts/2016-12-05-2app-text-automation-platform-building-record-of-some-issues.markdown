---
layout: post
title: "App自动化测试平台搭建记录之问题汇总"
date: 2016-12-05 18:25:38 +0800
commentps: true
categories: 
---
搭建测试平台及写测试case时，遇到了一些问题，现在统一做个汇总，记录一下
<!-- more -->
###问题1： 测试数据的问题  
a,测试每个case时，都会产生测试数据，怎么处理测试数据  
b,有些case有上下文逻辑关系，如果第一个case没跑，直接跑第二个case，肯定是跑不通的，也就是说数据之间也有逻辑关系  
c,正常情况下做自动化测试肯定是在生产环境或者类似生产环境上做测试，肯定不能在测试环境上做，因为要保证现在测试的东西跟生产环境是一模一样的，但是生产环境上测试肯定会产生很多的测试数据  

解决方法：  
以上问题解决方案是首先把现有的名个服务全部单独部署一套（这个代价有点大），这个环境是介于测试和生产之间，也就是说代码是生产稳定版本，数据库是备份生产的数据库下来（相当于是一个预发环境），这样就保证app的版本跟生产是一样的，只是数据不一样而已；因为数据库不是生产的，所以里面数据我们可以随意删减，每天跑测试case时，我会写一个脚本，把之前的测试数据全部删除，再自动增加一个逻辑上的测试数据上去，这样能保证在每天跑测试case时，数据库是干净的，而且各case都能保证之间的逻辑数据能衔接，现在的做法是每个跑case之前，定时跑一个脚本，删除一些测试数据，及增加一些逻辑数据，但是如果case多了，脚本复杂，其实很难确保删除处理完后，保证case能顺利通过，最好的办法应该是跑每个case之前把数据删除及增加逻辑数据，而不是一起跑，这样就能确保每个case的通过率。  
###问题2: 如何测试H5、hybrid app
app里面混入h5页面很正常，在一个即有原生页面又有H5页面，应该要怎么测试呢。appium其中的一个理念就是你不能为了测试应用而修改应用。为了符合这个方法学，我们可以使用 Selenium 测试传统 web 应用的方法来测试混合 web 应用 (比如，iOS 应用里的元素 "UIWebView" )，这是有可能的。这里会有一些技术性的复杂，Appium 需要知道你是想测试原生部分呢还是web部分。幸运的是，我们还能遵守 WebDriver 的协议。详细文档可以看[这里](https://github.com/appium/appium/blob/master/docs/en/advanced-concepts/hybrid.md)
appium会调用 GET session/:sessionId/contexts 接口来获得当前有context信息， 也可以使用POST session/:sessionId/context来访问当前的context可以具体看下代码
```ruby
查看当前页面有多少个contexts
pry(main)> available_contexts
[
    [0] "NATIVE_APP",
    [1] "WEBVIEW_49"
]

查看当前的context
[2] pry(main)> current_context
"NATIVE_APP"

转换context
[3] pry(main)> webview = available_contexts.last
"WEBVIEW_49"
[4] pry(main)> set_context webview
nil

```
这里需要特殊处理下是的ios环境，当在真机上运行用例时，appium 无法直接访问 web 视图，所以我们需要通过 USB 线缆来建立连接。我们使用 ios-webkit-debugger-proxy建立连接。使用 brew 安装最新的 ios-webkit-debug-proxy。在终端运行一下命令:
```ruby
# 如果你没有安装 brew 的话，先安装 brew。
> ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go/install)"
> brew update
> brew install ios-webkit-debug-proxy
```

一旦安装好了， 你就可以启动代理：

```ruby
# 将udid替换成你的设备的udid。确保端口 27753 没有被占用
# remote-debugger 将会使用这个端口。
> ios_webkit_debug_proxy -c c2848f21a19fa7dd3e83d4153225b77e9930039c:27753 -d

```
还有一个问题就是appium 1.6以上版本需要xcode8打出来的ipa包，1.6以下请用xcode7打包


###未完待续。。。
