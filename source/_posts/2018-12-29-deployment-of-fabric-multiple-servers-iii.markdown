---
layout: post
title: "Fabric多台服务器的部署(三)"
date: 2018-12-29 16:48:58 +0800
comments: true
categories: 
---
#### 6、启动Fabric网络  
&emsp;完成上面的工作后，我们已经生成了Fabric所需要的证书和创世纪块等一些必要文件后，现在我们就需要配置docker文件，来启动Fabric网络，之前我们讲过需要启动的一些服务，现在在列一下，服务器一共有4台， 跑4个peer，4个ca, 4个kafka，3个zookeeper, 2个orderer，具体信息如下：  
1、Server1：  peer1、Ca1、 Kafka1、Zookeeper1、Orderer1  
2、Server2：  peer2、Ca2、 Kafka2、Zookeeper2、Orderer2  
3、Server3：   peer3、Ca3、 Kafka3、Zookeeper31  
4、Server4：   peer4、Ca4、 Kafka4  
现在我们需要在各台服务器上启动这些服务，需要说明一下，我们需要先启动zookeeper，再启动kafka，然后启动orderer和peer，这些是有先后顺序的，因为各服务之间有依赖关系，不按顺序启动，最后可能各服务之间的通讯有问题，下面依次来启动这些服务。  
<!-- more -->
#####  6.1、启动Zookeeper集群
###### 6.1.1、配置docker-zookeeper.yaml文件，
&emsp;之前我们假设已经把trace_redwine项目push到远程仓库上，现在我们需要在四台服务器上分别把代码pull下来，进入/artifacts文件夹来配置docker的配置文件。  
&emsp;因为我们前三台服务器都需要docker-zookeeper.yaml文件，所以为了方便配置，我们不防生成三个zookeeper的配置文件，分别用于三台服务器配置上，依次命名如下：docker-zookeeper0.yaml、docker-zookeeper1.yaml、docker-zookeeper2.yaml，下面是docker-zookeeper0.yaml的配置如下：
```
version: '2'

services:

  zookeeper0:
    image: hyperledger/fabric-zookeeper
    container_name: zookeeper0
    hostname: zookeeper0
    restart: always
    environment:
      - ZOO_MY_ID=1
      - ZOO_SERVERS=server.1=zookeeper0:2888:3888 server.2=zookeeper1:2888:3888 server.3=zookeeper2:2888:3888
    ports:
      - 2181:2181
      - 2888:2888
      - 3888:3888
    extra_hosts:
      - "kafka0:10.99.22.103"
      - "kafka1:10.99.22.109"
      - "kafka2:10.99.22.101"
      - "kafka3:10.99.22.121"
      - "zookeeper0:0.0.0.0"
      - "zookeeper1:10.99.22.109"
      - "zookeeper2:10.99.22.101"
```
上面配置的参数有几点说明下：  
1、image是指定使用的镜像名称。  
2、container_name是启动容器时的名字。  
3、environment是设置Compose主机上对应的环境变量  
4、ports是暴露端口信息，格式是 宿主端口：容器端口 (HOST:CONTAINER)，  
5、extra_hosts是指定额外的 host 名称映射信息，当节点启动后，服务容器会在 /etc/hosts   文件中添加上面的配置的IP和名称，上面有配置会在/etc/hosts文件中添加如下配置。
```
10.99.22.103 kafka0
10.99.22.109 kafka1
10.99.22.101kafka2
10.99.22.121 kafka3
0.0.0.0 zookeeper0
10.99.22.109 zookeeper1
10.99.22.101 zookeeper2
```
6、ZOO_MY_ID是指zookeeper当前服务节点的id， 这个id值是一定需要填的，且唯一的，值的范围在1~255之间。  
10、ZOO_SERVERS是zookeeper集群服务器的列表。  

docker-zookeeper1.yaml、docker-zookeeper2.yaml文件配置整体不变，只需要修改一些变量名、环境变量就可以，docker-zookeeper1.yaml的配置文件是:
```
version: '2'

services:

  zookeeper0:
    image: hyperledger/fabric-zookeeper
    container_name: zookeeper1
    hostname: zookeeper1
    restart: always
    environment:
      - ZOO_MY_ID=2
      - ZOO_SERVERS=server.1=zookeeper0:2888:3888 server.2=zookeeper1:2888:3888 server.3=zookeeper2:2888:3888
    ports:
      - 2181:2181
      - 2888:2888
      - 3888:3888
    extra_hosts:
      - "kafka0:10.99.22.103"
      - "kafka1:10.99.22.109"
      - "kafka2:10.99.22.101"
      - "kafka3:10.99.22.121"
      - "zookeeper0:10.99.22.103"
      - "zookeeper1:10.99.22.109"
      - "zookeeper2:10.99.22.101"
```
需要注意的是我们现在是在多台服务器上，所以每个zookeeper服务的端口可以一样，因为不在一台服务器上，如果我们在本地上跑多个zookeeper，那么这里的端口是需要修改的；docker-zookeeper2.yaml的文件与上面类似，这里不再列出。
###### 6.1.2、启动Zookeeper
现在我们分别登录server1,server2,server3三台服务器，来启动这三个zookeeper服务，进入项目的/artifacts文件夹，找到每台服务器对应的docker zookeeper配置文件，使用docker-compose来启动
```
#启动zookeeper0
docker-compose -f docker-zookeeper0.yaml up -d
```
启动之后的输出
```
Starting zookeeper0 ... done
```
如果需要看启动的输出日志，则在上面启动命令后不要加 -d, 这样可以直接在启动的时候查看日志输出
```
docker-compose -f docker-zookeeper0.yaml up
```
server2,server3的启动跟上面一样，命令分别如下
```
#启动zookeeper1
docker-compose -f docker-zookeeper1.yaml up -d

#启动zookeeper2
docker-compose -f docker-zookeeper2.yaml up -d
```
启动成功后，可以使用docker命令查看运行状态
```
docker ps -a
```
可以看到zookeeper0是正在运行的状态
```
CONTAINER ID        IMAGE                                                                                                          COMMAND                  CREATED             STATUS              PORTS                                                                                                                                               NAMES
980c8bb7190d        hyperledger/fabric-zookeeper                                                                                   "/docker-entrypoint.…"   7 days ago          Up 7 days           0.0.0.0:2181->2181/tcp, 0.0.0.0:2888->2888/tcp, 0.0.0.0:3888->3888/tcp, 0.0.0.0:32773->2181/tcp, 0.0.0.0:32772->2888/tcp, 0.0.0.0:32771->3888/tcp   zookeeper0
```
#####  6.2、启动Kafka集群
###### 6.2.1、配置docker-kafka.yaml文件
因为我们有4个kafka服务，所以我们可以生成4个配置文件来启动这4个服务，配置文件分别如下docker-kafka0.yaml、docker-kafka1.yaml、docker-kafka2.yaml、docker-kafka3.yaml。
docker-kafka0.yaml的配置文件如下：
```
version: '2'

services:

  kafka0:
    image: hyperledger/fabric-kafka
    container_name: kafka0
    hostname: kafka0
    restart: always
    environment:
      - KAFKA_BROKER_ID=0
      - KAFKA_MIN_INSYNC_REPLICAS=2
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper0:2181,zookeeper1:2181,zookeeper2:2181
      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_LOG_RETENTION_MS=-1
      - KAFKA_LOG.DIRS=/var/hyperledger/production/kafka0/kafka-logs
    volumes:
       - ../mount/kafka0/var/hyperledger/production:/var/hyperledger/production/kafka0/kafka-logs
    ports:
      - 9092:9092
    extra_hosts:
      - "kafka0:10.99.22.103"
      - "kafka1:10.99.22.109"
      - "kafka2:10.99.22.101"
      - "kafka3:10.99.22.121"
      - "zookeeper0:10.99.22.103"
      - "zookeeper1:10.99.22.109"
      - "zookeeper2:10.99.22.101"
```
说明几点：  
1、KAFKA_BROKER_ID是BROKER的唯一标识，不可重复，且非负整数。    
2、KAFKA_ZOOKEEPER_CONNECT是指向zookeeper节点的集合。  
3、volumes是docker里数据卷所挂载路径设置，可以设置宿主机路径 （HOST:CONTAINER） 或加上访问模式（HOST:CONTAINER:ro）。    

docker-kafka1.yaml、docker-kafka2.yaml、docker-kafka3.yaml的配置基本与上面的一样，只要修改一下KAFKA_BROKER_ID参数就可以，如docker-kafka1.yaml的配置文件是：
```
version: '2'

services:

  kafka1:
    image: hyperledger/fabric-kafka
    container_name: kafka1
    hostname: kafka1
    restart: always
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_MIN_INSYNC_REPLICAS=2
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper0:2181,zookeeper1:2181,zookeeper2:2181
      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_LOG_RETENTION_MS=-1
      - KAFKA_LOG.DIRS=/var/hyperledger/production/kafka0/kafka-logs
    volumes:
       - ../mount/kafka0/var/hyperledger/production:/var/hyperledger/production/kafka0/kafka-logs
    ports:
      - 9092:9092
    extra_hosts:
      - "kafka0:10.99.22.103"
      - "kafka1:10.99.22.109"
      - "kafka2:10.99.22.101"
      - "kafka3:10.99.22.121"
      - "zookeeper0:10.99.22.103"
      - "zookeeper1:10.99.22.109"
      - "zookeeper2:10.99.22.101"
```
docker-kafka2.yaml、docker-kafka3.yaml按照上面改就行，这里不一一列出。
###### 6.2.2、启动Kafka集群
&emsp;现在我们分别登录4台服务器上，进入/artifacts项目路径下，找到刚才我们生成的那些docker-kafka.yaml的配置文件，使用命令依次启动各服务器上的kafka服务
```
#启动kafka0
docker-compose -f docker-kafka0.yaml up -d
```
server1，server2，server3启动方法和上面一样
```
#启动kafka1
docker-compose -f docker-kafka1.yaml up -d

#启动kafka2
docker-compose -f docker-kafka2.yaml up -d

#启动kafka3
docker-compose -f docker-kafka3.yaml up -d
```
最后可以在各服务器上使用docker 命令查看是否启动成功
```
docker ps -a

CONTAINER ID        IMAGE                                                                                                          COMMAND                  CREATED             STATUS              PORTS                                                                                                                                               NAMES
4bde08ae4413        hyperledger/fabric-kafka                                                                                       "/docker-entrypoint.…"   7 days ago          Up 7 days           0.0.0.0:9092->9092/tcp, 9093/tcp, 0.0.0.0:32774->9092/tcp                                                                                           kafka0
```
#####  6.3、启动Orderer集群
###### 6.3.1、配置docker-orderer.yaml文件
 &emsp;orderer服务我们只有server1,server2上有，所以我们只需要两个启动配置文件就行，命名如下docker-orderer0.yaml,docker-orderer1.yaml，
其中docker-orderer0.yaml的配置文件如下：
```
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

version: '2'

services:

  orderer0.xuyao.com:
      container_name: orderer0.xuyao.com
      image: hyperledger/fabric-orderer:latest
      environment:
        - ORDERER_GENERAL_LOGLEVEL=debug
        - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
        - ORDERER_GENERAL_GENESISMETHOD=file
        - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
        - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
        - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/crypto/orderer/msp
        - ORDERER_GENERAL_TLS_ENABLED=true
        - GODEBUG=netdns=go
        - ORDERER_GENERAL_TLS_PRIVATEKEY=/etc/hyperledger/crypto/orderer/tls/server.key
        - ORDERER_GENERAL_TLS_CERTIFICATE=/etc/hyperledger/crypto/orderer/tls/server.crt
        - ORDERER_GENERAL_TLS_ROOTCAS=[/etc/hyperledger/crypto/orderer/tls/ca.crt, /etc/hyperledger/crypto/peerOrg1/tls/ca.crt, /etc/hyperledger/crypto/peerOrg2/tls/ca.crt, /etc/hyperledger/crypto/peerOrg3/tls/ca.crt, /etc/hyperledger/crypto/peerOrg4/tls/ca.crt]
        - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
        - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
        - ORDERER_KAFKA_VERBOSE=true
      working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderers
      command: orderer
      ports:
        - 7050:7050
      volumes:
        - ./channel:/etc/hyperledger/configtx
        - ./channel/crypto-config/ordererOrganizations/xuyao.com/orderers/orderer0.xuyao.com/:/etc/hyperledger/crypto/orderer
        - ./channel/crypto-config/peerOrganizations/org1.xuyao.com/peers/peer0.org1.xuyao.com/:/etc/hyperledger/crypto/peerOrg1
        - ./channel/crypto-config/peerOrganizations/org2.xuyao.com/peers/peer0.org2.xuyao.com/:/etc/hyperledger/crypto/peerOrg2
        - ./channel/crypto-config/peerOrganizations/org3.xuyao.com/peers/peer0.org3.xuyao.com/:/etc/hyperledger/crypto/peerOrg3
        - ./channel/crypto-config/peerOrganizations/org4.xuyao.com/peers/peer0.org4.xuyao.com/:/etc/hyperledger/crypto/peerOrg4
        - ../mount/orderer1/var/hyperledger/production:/var/hyperledger/production
      extra_hosts:
        - "kafka0:10.99.22.103"
        - "kafka1:10.99.22.109"
        - "kafka2:10.99.22.101"
        - "kafka3:10.99.22.121"
```
说明：  
1、ORDERER_GENERAL_TLS_PRIVATEKEY是为服务器证书设置私钥文件的位置。  
2、ORDERER_GENERAL_TLS_CERTIFICATE是为TLS设置服务器证书的位置。  
3、ORDERER_GENERAL_TLS_ROOTCAS是为TLS设置根证书的位置。  

docker-orderer1.yaml的配置文件基本与docker-orderer1.yaml差不多，如下：
```

version: '2'

services:

  orderer1.xuyao.com:
      container_name: orderer1.xuyao.com
      image: hyperledger/fabric-orderer:latest
      environment:
        - ORDERER_GENERAL_LOGLEVEL=debug
        - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
        - ORDERER_GENERAL_GENESISMETHOD=file
        - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
        - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
        - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/crypto/orderer/msp
        - ORDERER_GENERAL_TLS_ENABLED=true
        - GODEBUG=netdns=go
        - ORDERER_GENERAL_TLS_PRIVATEKEY=/etc/hyperledger/crypto/orderer/tls/server.key
        - ORDERER_GENERAL_TLS_CERTIFICATE=/etc/hyperledger/crypto/orderer/tls/server.crt
        - ORDERER_GENERAL_TLS_ROOTCAS=[/etc/hyperledger/crypto/orderer/tls/ca.crt, /etc/hyperledger/crypto/peerOrg1/tls/ca.crt, /etc/hyperledger/crypto/peerOrg2/tls/ca.crt, /etc/hyperledger/crypto/peerOrg3/tls/ca.crt, /etc/hyperledger/crypto/peerOrg4/tls/ca.crt]
        - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
        - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
        - ORDERER_KAFKA_VERBOSE=true
      working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderers
      command: orderer
      ports:
        - 7050:7050
      volumes:
        - ./channel:/etc/hyperledger/configtx
        - ./channel/crypto-config/ordererOrganizations/xuyao.com/orderers/orderer1.xuyao.com/:/etc/hyperledger/crypto/orderer
        - ./channel/crypto-config/peerOrganizations/org1.xuyao.com/peers/peer0.org1.xuyao.com/:/etc/hyperledger/crypto/peerOrg1
        - ./channel/crypto-config/peerOrganizations/org2.xuyao.com/peers/peer0.org2.xuyao.com/:/etc/hyperledger/crypto/peerOrg2
        - ./channel/crypto-config/peerOrganizations/org3.xuyao.com/peers/peer0.org3.xuyao.com/:/etc/hyperledger/crypto/peerOrg3
        - ./channel/crypto-config/peerOrganizations/org4.xuyao.com/peers/peer0.org4.xuyao.com/:/etc/hyperledger/crypto/peerOrg4
        - ../mount/orderer1/var/hyperledger/production:/var/hyperledger/production
      extra_hosts:
        - "kafka0:10.99.22.103"
        - "kafka1:10.99.22.109"
        - "kafka2:10.99.22.101"
        - "kafka3:10.99.22.121"
```
###### 6.3.2、启动Orderer集群
 &emsp;我们分别在server1,server2上启动orderer，登录各服务器，进入各自的配置文件目录下，使用如下命令启动orderer
```
#启动orderer0
docker-compose -f docker-orderer0.yaml up -d

#启动orderer1
docker-compose -f docker-orderer1.yaml up -d
```
使用docker命令查看orderer状态
```
docker ps -a

CONTAINER ID        IMAGE                                                                                                          COMMAND                  CREATED             STATUS              PORTS                                                                                                                                               NAMES
806f354f4e62        hyperledger/fabric-orderer:latest                                                                              "orderer"                7 days ago          Up 7 days           0.0.0.0:7050->7050/tcp                                                                                                                              orderer1.mbasechain.com
```
#####  6.4、启动Peer集群
###### 6.4.1、配置docker-peer.yaml文件
 &emsp;peer节点在四台服务器上都有，所以我们需要生成4个配置文件分别叫docker-peer0.yaml, docker-peer1yaml,docker-peer2.yaml,docker-peer3.yaml，下面对docker-peer0.yaml作配置
```
version: '2'

services:

  couchdb:
    container_name: couchdb
    image: hyperledger/fabric-couchdb:latest
    ports:
      - "5984:5984"

  peer0.org1.xuyao.com:
    container_name: peer0.org1.xuyao.com
    image: hyperledger/fabric-peer:latest
    environment:
      - CORE_PEER_ID=peer0.org1.xuyao.com
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_ADDRESS=peer0.org1.xuyao.com:7051
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
    extra_hosts:
      - "orderer0.mbasechain.com:10.99.22.103"
      - "orderer1.mbasechain.com:10.99.22.109"
```
说明：  
1、CORE_PEER_ID是当前peer节点的id。  
2、CORE_PEER_LOCALMSPID是当前peer节点服务的成员服务提供者id。  
3、CORE_PEER_ADDRESS是当前peer节点在Fabric网络中的地址。  

docker-peer1.yaml,docker-peer2.yaml,docker-peer3.yaml文件配置和docker-peer0.yaml基本一样，修改一下主要的参数就行，这里不一一列出

###### 6.4.2、启动Peer集群
&emsp; 我们4台服务器上都有peer节点，所以我们现在分别登录这4台服务器，把刚才的配置文件更新好后，就可以依次启动这4个peer节点了。
```
#启动peer0
docker-compose -f docker-peer0.yaml up -d 

#启动peer1
docker-compose -f docker-peer1.yaml up -d 

#启动peer2
docker-compose -f docker-peer2.yaml up -d 

#启动peer3
docker-compose -f docker-peer3.yaml up -d 
```
启动成功后，我们再通过docker命令查看peer运行状态
```
docker ps -a

#运行状态
CONTAINER ID        IMAGE                                                                                                          COMMAND                  CREATED             STATUS              PORTS                                                                                                                                               NAMES
7a2582136861        hyperledger/fabric-peer:latest                                                                                 "peer node start"        7 days ago          Up 7 days           0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp                                                                                                      peer0.org1.mbasechain.com
```
#####  6.5、启动Ca集群
###### 6.5.1、配置docker-ca-org.yaml文件
&emsp; ca在4台服务器也都有，所以我们需要生成4个文件来分别配置4个ca，现生成4个docker配置文件为 docker-ca-org0.yaml,docker-ca-org1.yaml,docker-ca-org2.yaml,docker-ca-org3.yaml,配置如下
```
version: '2'

services:

  ca.org1.xuyao.com:
    container_name: ca
    image: hyperledger/fabric-ca:latest
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.org1.xuyao.com
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.ORG_NAME-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/9af18a6c9dd8f34db73a70c1d066e6948216db534af9e4f89029158089d56cf1_sk
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.xuyao.com-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/9af18a6c9dd8f34db73a70c1d066e6948216db534af9e4f89029158089d56cf1_sk
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start -b admin:adminpw -d'
    volumes:
      - ./channel/crypto-config/peerOrganizations/org1.xuyao.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.org1.xuyao.com
```
其它三个配置文件根据ca的名字不同，改下一些参数名字就行，将这些文件配置好后，提交到远程仓库
###### 6.5.2、启动Ca集群
&emsp; 现在我们分别登录4台服务器，把刚才修改的配置文件下来，准备启动ca服务，命令如下：
```
#启动ca-org0
docker-compose  -f docker-ca-org0.yaml up -d

#启动ca-org1
docker-compose  -f docker-ca-org1.yaml up -d

#启动ca-org2
docker-compose  -f docker-ca-org2.yaml up -d

#启动ca-org3
docker-compose  -f docker-ca-org3.yaml up -d
```
至此整个Fabric网络所需的节点都已经启动完了，我们可以使用命令来查看各个服务器所有正在运行的服务
```
#查看所以服务
docker ps -a

CONTAINER ID        IMAGE                                                                                                          COMMAND                  CREATED             STATUS              PORTS                                                                                                                                               NAMES
96529e278e54        hyperledger/fabric-ca:latest                                                                                   "sh -c 'fabric-ca-se…"   7 days ago          Up 7 days           0.0.0.0:7054->7054/tcp                                                                                                                              ca.org1.mbasechain.com
7a2582136861        hyperledger/fabric-peer:latest                                                                                 "peer node start"        7 days ago          Up 7 days           0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp                                                                                                      peer0.org1.mbasechain.com
49b5ee98950b        hyperledger/fabric-couchdb:latest                                                                              "tini -- /docker-ent…"   7 days ago          Up 7 days           4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                                                                                          couchdb
806f354f4e62        hyperledger/fabric-orderer:latest                                                                              "orderer"                7 days ago          Up 7 days           0.0.0.0:7050->7050/tcp                                                                                                                              orderer1.mbasechain.com
4bde08ae4413        hyperledger/fabric-kafka                                                                                       "/docker-entrypoint.…"   7 days ago          Up 7 days           0.0.0.0:9092->9092/tcp, 9093/tcp, 0.0.0.0:32774->9092/tcp                                                                                           kafka0
980c8bb7190d        hyperledger/fabric-zookeeper                                                                                   "/docker-entrypoint.…"   7 days ago          Up 7 days           0.0.0.0:2181->2181/tcp, 0.0.0.0:2888->2888/tcp, 0.0.0.0:3888->3888/tcp, 0.0.0.0:32773->2181/tcp, 0.0.0.0:32772->2888/tcp, 0.0.0.0:32771->3888/tcp   zookeeper0
```
可以看到docker里运行了zookeeper,kafka,orderer, peer, ca, couchdb6个服务，到这里整个Fabric启动工作就全部做好了，下篇文章将写安装channel和chaincode

##### 未完待续。。。

#### 参考资料
【HyperLedger Fabric 开发实战】
https://www.jianshu.com/p/040a09e2098e  
https://yeasy.gitbooks.io/docker_practice/data_management/volume.html

