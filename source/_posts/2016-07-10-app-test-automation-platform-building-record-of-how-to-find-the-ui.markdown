---
layout: post
title: "App自动化测试平台搭建记录之如何优雅地找到控件"
date: 2016-07-10 22:22:44 +0800
comments: true
categories: 
---
这篇讲一下如果找到对应ios/android里的UI控件，以便我们快速地写测试代码，本系列文章未特殊提到使用什么语言时，默认使用ruby脚本语言。ios/android设备的UI控件有很多种，下面讲一下使用appium Inspector和appium ruby console(arc)来查找UI控件  
<!-- more -->
####Appium Inspector GUI
appium服务启动后，点击inspector,启动client GUI，如下图,GUI已帮我们获取到所有的Ui控件，并且有很多模拟操作，非常适合我们调试用，点击record，每个模拟操作都会生成相应的代码，而且代码还可以选择你需要的语言版本。
![android_detail](http://upload-images.jianshu.io/upload_images/1796624-3b07ee8a85aaeadd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![android_record](http://upload-images.jianshu.io/upload_images/1796624-c58b0c22fd15b56c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
我们具体定位到某个控件，可以看到这个控件的详细信息，比如我现在找到一个登录按钮，下面是这个登录按钮的信息
```ruby
#android某个控件的详细信息
content-desc: 
type: android.widget.Button
text: 登录
index: 1
enabled: true
location: {418, 1054}
size: {262, 89}
checkable: false
checked: false
focusable: true
clickable: true
long-clickable: false
package: com.yydys
password: false
resource-id: com.yydys:id/immediate_experience
scrollable: false
selected: false
xpath: //android.widget.LinearLayout[1]/android.widget.FrameLayout[1]/android.widget.LinearLayout[1]/android.widget.LinearLayout[1]/android.widget.LinearLayout[2]/android.widget.Button[2]

#ios某个控件下的详情信息
name: 登录
type: UIAButton
value: 
label: 登录
hint: 
enabled: true
visible: true
valid: true
location: {192.5, 575}
size: {167.5, 44}
xpath: //UIAApplication[1]/UIAWindow[2]/UIAScrollView[1]/UIAButton[2]
```
####appium ruby console(arc)  
[ruby_console](https://github.com/appium/ruby_console)是针对ruby打造的一个console端的gem，使用起来也很方便，直接在console里使用命令arc就能启动设备，不过他并不能读取我们在appium里面的配置信息，所以需要我们手动创建一个appium.txt文件，在此文件目录下面启动，下面的android/ios的appium.txt文件
```ruby
#android
[caps]
platformName = "Android"
platformVersion = '5.1'
app = "/Users/xuyao/Downloads/Yydys0627.apk"
deviceName = "Android Simulator"


#ios
[caps]
platformName = "ios"
platformVersion = '8.2'
app = "/Users/xuyao/Downloads/Cloudoc-Patient.app"
deviceName = "iPhone Simulator"

#启动arc
➜ ls
appium.txt
➜ arc
[1] pry(main)>
```
启动后，我们可以通过page方法查看当前页面所有的元素
```ruby
[4] pry(main)> page

android.widget.FrameLayout (1)
  id: android:id/content

android.support.v4.view.ViewPager (0)
  id: com.yydys:id/vp_welcome

android.widget.ImageView (0)
  id: com.yydys:id/wel_img

android.widget.Button (0)
  text: 逛一逛
  id: com.yydys:id/direct_access
  strings.xml: direct_access

android.widget.Button (1)
  text: 登录
  id: com.yydys:id/immediate_experience
  strings.xml: immediate_experience
               login
nil
```
如果觉得页面信息很多，找不到自己想要的控件，page方法后面可以带个参数class_name来过滤元素，如下
```ruby
[5] pry(main)> page :Button

android.widget.Button (0)
  text: 逛一逛
  id: com.yydys:id/direct_access
  strings.xml: direct_access

android.widget.Button (1)
  text: 登录
  id: com.yydys:id/immediate_experience
  strings.xml: immediate_experience
               login
nil
```
这样就能找到按钮的元素了
######find_element()方法  
根据上面的信息我们可以通过find_elment方法来查找android/ios下的xpath,模拟点击操作
```ruby
#android
find_element(xpath: "//android.widget.LinearLayout[1]/android.widget.FrameLayout[1]/android.widget.LinearLayout[1]/android.widget.LinearLayout[1]/android.widget.LinearLayout[2]/android.widget.Button[2]").click
#android我们还可以根据他的recource-id，来查找他的唯一控件
find_element(id: "com.yydys:id/immediate_experience").click
#ios
find_element(xpath: "//UIAApplication[1]/UIAWindow[2]/UIAScrollView[1]/UIAButton[2]").click
#当然你也可以通过class_name来查找，如果一个页面上有多个相同的class_name,会返回第一个tag
find_element(class_name: "android.widget.Button").click  #android
find_element(class_name: "UIAButton").click
```
######button_exact方法
有时候我们知道要找一个按钮，那么我们可以使用button_exact来直接查找一个button,button_exact方法会找到第一个配置到的元素
```ruby
[8] pry(main)> button_exact("登录")
#<Selenium::WebDriver::Element:0x5dffe05a97580bc2 id="3">
```
######textfield_exact()方法
根据给定的值找到EditText/TextFeild第一个元素
```ruby
[24] pry(main)> textfield_exact("com.yydys:id/login_input").type "9123"
true
```
######exists()方法
测试的时候一般需要判断某个元素是否存在，可以使用exists
```ruby
[40] pry(main)> exists{button_exact("com.yydys:id/login_btn")}
true
```
以上就是一些比较常用的方法，以后用到再更新一些新的方法  
#####参考资料
https://github.com/appium/ruby_lib/blob/master/docs/android_docs.md  
https://github.com/appium/ruby_lib/blob/master/docs/ios_docs.md  
https://ruby-china.org/topics/30154
