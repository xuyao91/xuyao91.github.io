---
layout: post
title: "about sso in rails"
date: 2016-04-06 16:19:32 +0800
comments: true
categories: 
---
###前言
目前业务越来越多，很多业务可能需要拆分成不同的应用，不同的应用之间如何互享用户信息，这时就需要用到单点登录(Single Sign On），简称为 SSO。  

这里主要针对doorkeeper作讲解  
<!-- more -->

![查看图](http://i2.piimg.com/285bc33bd4eab4b1.png)
###需要用到的插件
*   [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper)  ([Oauth 2.0](http://oauth.net/2/)协议的认证授权服务)
*   Grape (提供 Resource Server APi)
*   Devise (提供登录，暂时可能用不到)


###需要用到的角色
*   Resource Owner (用户角色)
*   Clients (第三方角色)
*   Authorization Server (授权服务) 
*   Resource Server (用户信息读写服务,api形式)  
详情可查看[RFC6749](https://github.com/jeansfish/RFC6749.zh-cn/blob/master/Section01/1.1.md)


###Authorization Server构建
安装doorkeeper及初始模型的建立
```ruby
gem 'doorkeeper' 
#bundle install
rails generate doorkeeper:install
rails generate doorkeeper:migration
rake db:migrate
```
配置doorkeeper来提供resource_owner的model及授权

```ruby
Doorkeeper.configure do
  resource_owner_authenticator do
    Patient.where(id: session[:patient_id], authentication_token: session[:patient_auth_token]).first || redirect_to(login_url)
  end
end

```
上面操作会生成下面几张表

*   oauth_applications  第三方(clients)表
*   oauth_access_grants 用户授权code表
*   oauth_access_tokens 授权服务token表


###添加 Applications
授权前需要添加第三方应用的信息，doorkeeper提供了一个增加Clients的功能
[点击查看](http://localhost:3001/oauth/applications)

###Authorization Grant Code Flow
*    Resource Owner请求Client的Redirection URI 
*    Client转到Authorization Server
*    Resource Owner 确认授权
*    Authorization Server验证用户
*    Authorization Server返回Authorization Code
*    Resource Owner返回Redirection URI & Authorization Code
*    Client拿到Authorization Code请求Access Token

具体[点击查看](https://tools.ietf.org/ht，ml/draft-ietf-oauth-v2-22#section-4.1)

###获取Access Token
上面介绍了oauth 2的授权流程，得到Client信息后，就可以去Authorization Server获取Token,通过调用/oauth/token接口获得Token  
具体可以看  [模拟操作](http://localhost:3001/oauth/applications)

###Resource Server构建
使用grape作Resource Server的接口,暂时可能需要如下接口

1，获取用户信息  
2，修改用户信息  
查看swagger[模拟操作](http://localhost:3001/documentation/api_v1)



###说明及问题 
1. 把Authorization Server部署在哪里
>部署在主项目里面，这样可以省去Authorization Server的登录，因为App登录后，Authorization Server可以通过session获得用户信息
1. 目前只在App内授权，不提供外部登录
> 我们的其它应用都是App内的页面，而且在外部有些可能不能用，所以在App里授权
2. Api部分可以不做验证


####参考资料
[https://github.com/doorkeeper-gem/doorkeeper](https://github.com/doorkeeper-gem/doorkeeper)  
[https://github.com/sethherr/grape-doorkeeper](https://github.com/sethherr/grape-doorkeeper)  
[https://blog.yorkxin.org/posts/2013/10/10/oauth2-tutorial-grape-api-doorkeeper/](https://blog.yorkxin.org/posts/2013/10/10/oauth2-tutorial-grape-api-doorkeeper/)  
[http://tools.ietf.org/html/rfc6749](http://tools.ietf.org/html/rfc6749)   
[https://github.com/jeansfish/RFC6749.zh-cn](https://github.com/jeansfish/RFC6749.zh-cn)  
[https://ruby-china.org/topics/15396](https://ruby-china.org/topics/15396)
