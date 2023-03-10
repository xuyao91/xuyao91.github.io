---
layout: post
title: "安装frp内网穿透工具"
date: 2023-03-10 10:30:17 +0800
comments: true
categories: 
---

#### 1、需求背景
客户现场有几台服务器，是局域网的，因为要经常维护，显示每次去跑现场肯定是不合适的，为了便于维护和管理，可以装一个内网穿透工具--frp

<!--more-->

#### 2、安装

1. 由于frp是golang语言写的，所以只要下载编译好的源码，直接跑就行，下载源码
```
下载最新版本的frp
wget https://github.com/fatedier/frp/releases/download/v0.45.0/frp_0.45.0_linux_amd64.tar.gz
```
解压gz包
```
tar -zxvf  frp_0.45.0_linux_amd64.tar.gz
```

2. 配置frp，frp需要有一个公网ip，通过公网ip转发到要访问的局域网上，所以整个过程分为服务端和客户端，服务端是指有公网ip的那个服务器，客户端是需要访问的那台局域网  
服务端的配置文件 
```
打开配置文件
vim frps.ini

[common]
bind_port = 8700 #配置需要转发的端口
```
客户端配置文件
```
vim frpc.ini
[common]
server_addr = 106.14.xxx.161 #公网服务器的IP
server_port = 8700 #公网服务端配置的端口

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 8600 #公网服务器的登录端口，
```

3. 启动服务端和客户端
服务端启动
```
./frps -c frps.ini
```
客户端启动
```
./frpc -c frpc.ini
```
这样整个过种就可以了,可以试下
```
通过端口登录公网的ip,实际是登录了局域网
ssh -oPort=8600 root@106.14.xxx.161
```
4.使用systemd启动frp,正常工作中，一般frp都要一直开着的，包括开机启动等，所以可以使用systemd来守护进程，
安装systemd
```
# yum
yum install systemd
# apt
apt install systemd
```
创建frps的配置文件
```
vim /etc/systemd/system/frps.service
```
配置文件内容
```
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /path/to/frps -c /path/to/frps.ini 
# ExecStart = /path/to/frpc -c /path/to/frpc.ini

[Install]
WantedBy = multi-user.target
```
使用systemd 命令，管理 frps。
```
# 启动frp
systemctl start frps
# 停止frp
systemctl stop frps
# 重启frp
systemctl restart frps
# 查看frp状态
systemctl status frps
```
配置开机自动启动frps
```
systemctl enable frps
```

查看frps启动日志
```
journalctl -u frps.service
```
#### 3、身份认证

frp还可以支持身份验证，有三种模式，token 和 oidc，默认为 token。  
简单给个token身份验证的方式，这个也是最常用的。
```
# frps.ini 服务端
[common]
bind_port = 8700
authentication_method = token
token = QqQ27WRVKGUNxxxx


# frpc.ini 客户端
[common]
server_addr = 106.xx.xx.xx
server_port = 8700
authentication_method = token
token = QqQ27WRVKGUNxxxx
```

#### 4、配置自定义域名访问内网的Web服务
可以使用vhost_http_port用来监听8080端口
```
#frps.ini 服务端
[common]
bind_port = 7000
vhost_http_port = 8800


#frpc.ini 客户端
[web]
type = http
local_port = 8800
custom_domains = xxx.1nongfu.com
```
这样就可以通过浏览器访问http://xxx.1nongfu.com:8800 即可访问到处于内网机器上 8800 端口的服务，如果是静态资源可能还需要用nginx做个代理。  


frp还有很多功能，这个是最简单的例子

#### 参考资料

1. https://gofrp.org/docs/overview/
2. https://gofrp.org/docs/setup/systemd/
