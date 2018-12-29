---
layout: post
title: "Fabric多台服务器的部署(五)"
date: 2018-12-29 16:57:34 +0800
comments: true
categories: 
---
#### 8、Fabric 浏览器的部署
###### 8.1 环境准备
1、nodejs安装，在第1节里已经说过node的安装，需要注意的是node版本应该是v9.x以下。  
2、PostgreSQL的安装，浏览器会把监听到的数据存储在pg里，所以需要安装pg数据。  
<!-- more -->
ubuntu使用直接安装
```
sudo apt update
sudo apt install postgresql postgresql-contrib
```
安装好后，可以使用cli查看一下
```
#切换用户
sudo -i -u postgres
#进入控制台
psql
#退出
postgres=# \q
```
3、安装Jq，jq是一个轻量级且灵活的命令行JSON处理器，安装很简单，见官网[https://stedolan.github.io/jq/download/](https://stedolan.github.io/jq/download/)

##### 8.2、下载浏览器源码
&emsp; 在github上下载fabric 浏览器的源码，[https://github.com/hyperledger/blockchain-explorer](https://github.com/hyperledger/blockchain-explorer)，把源码拉下来，根据自己的需要选择不用的版本，我目前使用的是1.1版本，所以我切换到对应的1.1分支上。
##### 8.3、修改配置
###### 8.3.1 配置postgresql数据库
&emsp; 进入 app/persistence/postgreSQL/db/文件夹，找到pgconfig.json配置文件，修改连接pg数据库的配置，默认配置为
```
{
  "pg": {
    "host": "127.0.0.1",
    "port": "5432",
    "database": "fabricexplorer",
    "username": "hppoc",
    "passwd": "password"
  }
}
```
修改支运行createdb.sh脚本文件，创建浏览器所需的数据库和表名。
###### 8.3.2、配置启动abric浏览器文件
进入 blockchain-explorer/app/platform/fabric，找到对应的config.json文件，配置对应的fabric网络信息，包括节点的请求接口，及监听接口，不同浏览器版本这个配置文件不太一样，1.1版本的配置文件如下：
```
{
  "network-config": {
    "org1": {
      "name": "peerOrg1",
      "mspid": "Org1MSP",
      "peer1": {
        "requests": "grpcs://localhost:7051",
        "events": "grpcs://localhost:7053",
        "server-hostname": "peer0.org1.mbasechain.com",
        "tls_cacerts": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org1.mbasechain.com/peers/peer0.org1.mbasechain.com/tls/ca.crt"
      },
      "admin": {
        "key": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org1.mbasechain.com/users/Admin@org1.mbasechain.com/msp/keystore",
        "cert": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org1.mbasechain.com/users/Admin@org1.mbasechain.com/msp/signcerts"
      }
    },
    "org2": {
      "name": "peerOrg2",
      "mspid": "Org2MSP",
      "peer1": {
        "requests": "grpcs://localhost:7056",
        "events": "grpcs://localhost:7058",
        "server-hostname": "peer0.org2.mbasechain.com",
        "tls_cacerts": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org2.mbasechain.com/peers/peer0.org2.mbasechain.com/tls/ca.crt"
      },
      "admin": {
        "key": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org2.mbasechain.com/users/Admin@org2.mbasechain.com/msp/keystore",
        "cert": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org2.mbasechain.com/users/Admin@org2.mbasechain.com/msp/signcerts"
      }
    },
    "org3": {
      "name": "peerOrg3",
      "mspid": "Org3MSP",
      "peer1": {
        "requests": "grpcs://localhost:8051",
        "events": "grpcs://localhost:8053",
        "server-hostname": "peer0.org3.mbasechain.com",
        "tls_cacerts": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org3.mbasechain.com/peers/peer0.org3.mbasechain.com/tls/ca.crt"
      },
      "admin": {
        "key": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org3.mbasechain.com/users/Admin@org3.mbasechain.com/msp/keystore",
        "cert": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org3.mbasechain.com/users/Admin@org3.mbasechain.com/msp/signcerts"
      }
    },
    "org4": {
      "name": "peerOrg4",
      "mspid": "Org4MSP",
      "peer1": {
        "requests": "grpcs://localhost:8056",
        "events": "grpcs://localhost:9058",
        "server-hostname": "peer0.org4.mbasechain.com",
        "tls_cacerts": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org4.mbasechain.com/peers/peer0.org4.mbasechain.com/tls/ca.crt"
      },
      "admin": {
        "key": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org4.mbasechain.com/users/Admin@org4.mbasechain.com/msp/keystore",
        "cert": "/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/peerOrganizations/org4.mbasechain.com/users/Admin@org4.mbasechain.com/msp/signcerts"
      }
    }
  },
  "channel": "mychannel",
  "orderers":[
    {
      "mspid": "OrdererMSP",
      "server-hostname":"orderer1.mbasechain.com",
      "requests":"grpcs://localhost:7050",
      "tls_cacerts":"/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/ordererOrganizations/mbasechain.com/orderers/orderer1.mbasechain.com/tls/ca.crt"
    },
    {
      "mspid": "OrdererMSP",
      "server-hostname":"orderer2.mbasechain.com",
      "requests":"grpcs://localhost:8050",
      "tls_cacerts":"/Users/xuyao/Workspaces/trace_kingland/artifacts/channel/crypto-config/ordererOrganizations/mbasechain.com/orderers/orderer2.mbasechain.com/tls/ca.crt"
    }
  ],
  "keyValueStore": "/tmp/fabric-client-kvs",
  "configtxgenToolPath": "/Users/xuyao/Workspaces/goworkspace/src/blockchain/fabric/v1.2.0/fabric-samples/bin",
  "SYNC_START_DATE_FORMAT":"YYYY/MM/DD",
  "syncStartDate":"2018/01/01",
  "eventWaitTime": "30000",
  "license": "Apache-2.0",
  "version": "1.1"
}
```
配置各节点的信息及tls的ca证书，管理员的私钥及证书等信息。
###### 8.3.3、构建node项目
1、进入blockchain-explorer/ 下，使用npm install 安装项目所需package，  
2、再进入blockchain-explorer/app/test/下，使用npm install 安装测试项目里的所需包，再跑测试脚本  
```
#安装包
npm install

#运行测试脚本
npm run test
```
3、进入client/ 使用npm install 命令安装包
```
npm install

npm test -- -u --coverage

npm run build
```

##### 8.4、启动fabric浏览器
&emsp; 使用脚本./start.sh来启动项目，启动成功后，可以打开浏览器查看一下.
![浏览器截图](https://upload-images.jianshu.io/upload_images/1796624-c1112936dd3eabf3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中做了一些汉化。
##### 已完结

##### 参考资料
1、https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04  
2、https://github.com/hyperledger/blockchain-explorer  
3、https://stedolan.github.io/jq/download/  



