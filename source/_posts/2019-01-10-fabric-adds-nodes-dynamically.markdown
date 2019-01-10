---
layout: post
title: "Fabric动态增加节点（Peer）"
date: 2019-01-10 11:00:22 +0800
comments: true
categories: 
---
##### 前言
 &emsp; 在实际项目中，因为项目的需求变动，一般都会对fabric网络做一个修改，最常见的变动就是在现有的Fabric网络中增加一个节点，现在以之前搭建的fabric网络（版本1.1），balance-transfer为例做个介绍，整个过程其实也很简单，生成节点证书，增加新节点的docker配置文件并启动相应的服务，然后将新节点加到现有的channel当中，并在节点上安装智能合约（Chaincode），下午详情讲一下具体的操作步骤。
#### 1、修改crypto-config.yaml文件并生成对应的节点证书。
首先我们确认我们需要在Org2里增加节点，那么我们在crypto-config.yaml文件里找到对应Org2的配置，把Template字段里的count参数修改成2，意思就是在此组织下生成两个节点，配置文件如下
<!-- more -->
```
- Name: Org2
    Domain: org2.xuyao.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
```
修改好配置文件后，我们现在使用cryptogen来生成证书，在在artifacts/channel目录下执行：
```
./cryptogen extend --config=./crypto-config.yaml
```
这样我们就为新节点生成了证书，可以到Org2相应的目录上查看，发现已经新生成了一个节点，加上之前那个，一共有两个节点。
```
peer0.org2.mbasechain.com peer1.org2.mbasechain.com
```
#### 2、配置docker-compose文件并启动节点容器
接下来我们需要新增一个doceker-compose有配置文件，用来启动一个新节点容器，我们将新节点的配置文件全名为：Org2Peer1.yaml
```
version: '2'

services:

  couchdb:
    container_name: couchdb
    image: hyperledger/fabric-couchdb:latest
    ports:
      - "5984:5984"

  peer1.org1.xuyao.com:
    container_name: peer1.org1.xuyao.com
    image: hyperledger/fabric-peer:latest
    environment:
      - CORE_PEER_ID=peer1.org1.xuyao.com
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_ADDRESS=peer1.org1.xuyao.com:7051
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb:5984
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=artifacts_default
      - CORE_LOGGING_LEVEL=debug
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - GODEBUG=netdns=go
      # The following setting skips the gossip handshake since we are
      # are not doing mutual TLS
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/crypto/peer/msp
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/crypto/peer/tls/server.key
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/crypto/peer/tls/server.crt
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/crypto/peer/tls/ca.crt
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 7051:7051
      - 7053:7053
    volumes:
        - /var/run/:/host/var/run/
        - ./channel/crypto-config/peerOrganizations/org1.mbasechain.com/peers/peer0.org1.xuyao.com/:/etc/hyperledger/crypto/peer
        - ../mount/peer0.org1.xuyao.com/var/hyperledger/production:/var/hyperledger/production
    depends_on:
        - couchdb
```
这个配置文件跟之前的peer节点的配置一样，只要改下相应的参数即可，接下来我们就来启动这个节点。
```
docker-compose -f Org2Peer1.yaml up -d
```
启动过程也和之前的一样。
#### 3、修改network-config.json配置文件
启动节点后，我们需要配置network-config.json，让应用与节点之前通信，在peers那块增加一个通讯节点就可以
```
peer1.org2.xuyao.com:
    url: grpcs://127.0.0.1:10051
    eventUrl: grpcs://127.0.0.1:10053
    grpcOptions:
      ssl-target-name-override: peer1.org2.xuyao.com
    tlsCACerts:
      path: artifacts/channel/crypto-config/peerOrganizations/org4.mbasechain.com/peers/peer1.org2.xuyao.com/tls/ca.crt
```
修改配置后，我们启动balance-transfer,运行runApp.sh,将项目跑起来。
#### 4、将新节点加入到所需的Channel中
接下来我们需要将新节点加到某个Channel里，首先我们通过注册接口，在Org2里拿到用户的Token,再拿此Token请求中入Channel接口
```
curl -s -X POST \
  http://localhost:4000/channels/mychannel/peers \
  -H "authorization: Bearer $ORG2_TOKEN" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer1.org2.xuyao.com"]
}'
```
#### 5、安装Chaincode
加入channel后，peer3已经可以参与记账，但是不能指定该节点进行查询或交易，这时候需要发起请求安装chaincode：
```
curl -s -X POST \
    http://localhost:$PORT/chaincodes \
    -H "authorization: Bearer $ORG1_TOKEN" \
    -H "content-type: application/json" \
    -d '{
    "peers": ["peer1.org2.xuyao.com"],
    "chaincodeName":"mycc",
    "chaincodePath":"github.com/example_cc",
    "chaincodeVersion":"v0"
  }'
```
安装成功后指定新节点进行查询或交易操作，会自动生成该节点的chaincode镜像，并启动容器运行chaincode。在已有组织中新加节点的操作到这里就全部完成了！
