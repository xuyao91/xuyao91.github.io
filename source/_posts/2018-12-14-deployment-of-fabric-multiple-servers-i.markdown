---
layout: post
title: "Fabric多台服务器的部署(一)"
date: 2018-12-14 11:55:10 +0800
comments: true
categories: 
---
### 1、前言
  &emsp;Hyperledger Fabric是一个提供分布式账本解决方案的平台。Hyperledger Fabric由模块化架构支撑，并具备极佳的保密性、可伸缩性、灵活性和可扩展性。Hyperledger Fabric被设计成支持不同的模块组件直接拔插启用，并能适应在经济生态系统中错综复杂的各种场景。
 &emsp;目前Fabric在区块链溯源场景应用的挺多，农产品，奢侈品，艺术品，红酒等方面都看过到案例，区块链可以保存产品各个环节的数据，并上链，用户最后可能只需扫描一个二维码，就可以查出某个产品整个的生产过程，与中心化数据不同的是，区块链溯源项目数据一旦上链后不可更改，而且数据公开透明，任何一方都没有权利来更改它，同时每个环节需要为你上链的数据负责，如果哪天消费查出某个环节的数据是假的，由于账本数据是分布式的、不可更改的，大家只相信数据，商家也无法抵赖，最后商家只能认错，名誉扫地，所以区块链本质上是解决一个多方协作的一个信任问题，而对于一件事情需要多方协作，共同记账来一起完成的，Fabric作为联盟链，显然很符合这种需求。
<!-- more --> 
### 2、环境安装及服务器配置
##### 2.1 环境安装
在配置Fabric网络之前，需要安装一下相关环境，包括GO语言安装和Docker的安装
###### 2.1.1 GO语言安装
1、从官网下载它的安装包，地址：[https://golang.org/dl/](https://golang.org/dl/)，根据自己电脑或服务器不同系统下载对应的安装包。
2、下载后解压包
```
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
```
3、配置GO的环境变量
```
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/opt/gopath
```
最后执行source命令，使其生效
```
source profile
```
最后打印一下GO版本
```
go version
go version go1.10.1 darwin/amd64
```
**ubuntu系统下安装**
```
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt-get update
sudo apt-get install golang-go
```
设置环境变量
```
export GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```
不同系统安装略有不同，具体建议查看官方文档 [https://golang.org/doc/install](https://golang.org/doc/install)
##### 2.1.2 Docker、Docker-Compose的安装
docker安装不同系统也是有区别的，我是mac电脑，安装也相对简单，到docker[官网](https://www.docker.com/products/docker-desktop)下载一个包，直接安装就行，装好后可以查一下docker的版本，值得注意的是，mac电脑只安装docker就行，docker版本已经包括了compose和其它docker应用，所以无需再另行安装compose
```
docker version
#docker的版本信息
Client:
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        e68fc7a
 Built:             Tue Aug 21 17:21:31 2018
 OS/Arch:           darwin/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.1-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       e68fc7a
  Built:            Tue Aug 21 17:29:02 2018
  OS/Arch:          linux/amd64
  Experimental:     true
```

如果是Linux系统，需要安装docker和docker-compose，可以参考这里[https://yeasy.gitbooks.io/docker_practice/install/](https://yeasy.gitbooks.io/docker_practice/install/),针对不同的linux系统，都有安装方法，写的比较详细，也可以参考官网信息[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)  

**ubuntu系统安装docker、docker-compose**  


卸载旧版本的docker
```
$ sudo apt-get remove docker \
               docker-engine \
               docker.io
```
鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。
为了确认所下载软件包的合法性，需要添加软件源的 GPG 密钥。
```
$ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```
安装docker-ce
```
$ sudo apt-get update

$ sudo apt-get install docker-ce
```
安装docker-compose，在 Linux 上的也安装十分简单，从 [官方 GitHub Release](https://github.com/docker/compose/releases) 处直接下载编译好的二进制文件即可。
例如，在 Linux 64 位系统上直接下载对应的二进制包。
```
curl -L https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

##### 2.1.3 nodejs安装
 &emsp;因为我们项目用的是nodejs SDK，所以实际项目中还需要安装nodejs和npm来跑项目，因为Fabric目前只支持nodejs v8.4.0~v9.0.0的版本（2018-10），所以建议安装的时候指定一个固定的版本，比如v8.9.0，我用的是这个版本，还是很稳定的，当然也可以先下载一个nvm来管理nodejs的版本，不同项目需要不同的node版本，使用nvm可以管理不同的node版本。
1、安装nvm，执行下面命令下载安装脚本
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
#输出

=> Close and reopen your terminal to start using nvm or run the following to use it now:
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```
检查下是否安装成功
```
nvm --version
0.33.11
```
2、安装nodejs，使用nvm ls-remote查看远程可安装的node版本
```
nvm ls-remote 
v8.0.0
v8.1.0
v8.1.1
v8.1.2
v8.1.3
v8.1.4
v8.2.0
v8.2.1
v8.3.0
v8.4.0
v8.5.0
v8.6.0
v8.7.0
v8.8.0
v8.8.1
v8.9.0
```
找到自己想要安装的版本来安装nodejs
```
nvm install 8.9.0
```
使用node命令查看是否安装成功
```
node -v
v8.9.1
```
也可以使用nvm ls来查看当前环境有几个node版本
```
nvm ls

->       v8.9.1
         system
default -> 8.9.1 (-> v8.9.1)
node -> stable (-> v8.9.1) (default)
stable -> 8.9 (-> v8.9.1) (default)
iojs -> N/A (default)
lts/* -> lts/dubnium (-> N/A)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.15.0 (-> N/A)
lts/carbon -> v8.14.0 (-> N/A)
lts/dubnium -> v10.14.1 (-> N/A)
```
可以看到目前只安装了8.9.0，而且系统默认使用的就是8.9.0，如果想改变系统的默认版本，可以使用如下命令
```
nvm use 8.9.0
Now using node v10.13.0 (npm v6.4.1)
```
##### 2.1.4 Git工具安装
Git安装就简单了吧，做过开发的人应该都安装过，这里贴个安装教程，
[https://git-scm.com/book/en/v2/Getting-Started-Installing-Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git),跟着上面安装就行。


##### 2.2 服务器配置说明
 &emsp;说明一下服务器的整个情况，我申请了4台服务器， 做4个peer，两个orderer，具体的crypto-config.yaml配置如下：
```

OrdererOrgs:
  # ---------------------------------------------------------------------------
  # Orderer
  # ---------------------------------------------------------------------------
  - Name: Orderer
    Domain: xuyao.com
    # ---------------------------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
    # ---------------------------------------------------------------------------
    Specs:
      - Hostname: orderer1
      - Hostname: orderer2
# ---------------------------------------------------------------------------
# "PeerOrgs" - Definition of organizations managing peer nodes
# ---------------------------------------------------------------------------
PeerOrgs:
  # ---------------------------------------------------------------------------
  # Org1
  # ---------------------------------------------------------------------------
  - Name: Org1
    Domain: org1.xuyao.com
    EnableNodeOUs: true
    # ---------------------------------------------------------------------------
    # "Specs"
    # ---------------------------------------------------------------------------
    # Uncomment this section to enable the explicit definition of hosts in your
    # configuration.  Most users will want to use Template, below
    #
    # Specs is an array of Spec entries.  Each Spec entry consists of two fields:
    #   - Hostname:   (Required) The desired hostname, sans the domain.
    #   - CommonName: (Optional) Specifies the template or explicit override for
    #                 the CN.  By default, this is the template:
    #
    #                              "{{.Hostname}}.{{.Domain}}"
    #
    #                 which obtains its values from the Spec.Hostname and
    #                 Org.Domain, respectively.
    # ---------------------------------------------------------------------------
    # Specs:
    #   - Hostname: foo # implicitly "foo.org1.example.com"
    #     CommonName: foo27.org5.example.com # overrides Hostname-based FQDN set above
    #   - Hostname: bar
    #   - Hostname: baz
    # ---------------------------------------------------------------------------
    # "Template"
    # ---------------------------------------------------------------------------
    # Allows for the definition of 1 or more hosts that are created sequentially
    # from a template. By default, this looks like "peer%d" from 0 to Count-1.
    # You may override the number of nodes (Count), the starting index (Start)
    # or the template used to construct the name (Hostname).
    #
    # Note: Template and Specs are not mutually exclusive.  You may define both
    # sections and the aggregate nodes will be created for you.  Take care with
    # name collisions
    # ---------------------------------------------------------------------------
    Template:
      Count: 2
      # Start: 5
      # Hostname: {{.Prefix}}{{.Index}} # default
    # ---------------------------------------------------------------------------
    # "Users"
    # ---------------------------------------------------------------------------
    # Count: The number of user accounts _in addition_ to Admin
    # ---------------------------------------------------------------------------
    Users:
      Count: 1
  # ---------------------------------------------------------------------------
  # Org2: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: Org2
    Domain: org2.xuyao.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1

  # ---------------------------------------------------------------------------
  # Org3: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: Org3
    Domain: org3.xuyao.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1

  # ---------------------------------------------------------------------------
  # Org4: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: Org4
    Domain: org4.xuyao.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
```
每台服务器的运行大致如下  

1. server1: Org1, Peer1，Orderer1，Ca1，Kafka1, Zookeeper1 
2. server1:  Org2, Peer2，Orderer2，Ca2，Kafka2, Zookeeper2 
3. server3:  Org3, Peer3，Ca3，Kafka3, Zookeeper3  
4. server4:  Org4, Peer4，Ca4，Kafka4  
每台服务器一个Org组织，和一个节点， 其中两台服务器有Orderer服务，四台服务器都有Ca认证。
###### 未完待续.....


#####参考资料
https://golang.org/doc/install  
https://github.com/golang/go/wiki/Ubuntu
https://yeasy.gitbooks.io/docker_practice/install/  
https://github.com/docker/compose/releases
https://linuxize.com/post/how-to-install-node-js-on-centos-7  
https://git-scm.com/book/en/v2/Getting-Started-Installing-Git  
https://www.cnblogs.com/aberic/p/7527831.html  

