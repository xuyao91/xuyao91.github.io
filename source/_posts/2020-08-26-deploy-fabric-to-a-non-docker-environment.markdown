---
layout: post
title: "非Docker环境部署Fabric"
date: 2020-08-26 11:09:59 +0800
comments: true
categories: 
---

本文使用ubuntu 18作为系统，直接使用脚本启动Fabric网络，而非使用Docker容器来启动
### 1、安装环境  
**1.1、安装Ubuntu系统**  
到官网下载最新版的ubuntu镜像，我本地是使用虚拟机来安装ubuntu的，打开虚拟机，选择镜像文件，根据提示一路安装，
<!-- more -->
**1.2、安装Docker**  
因为chaincode最终还是要在docker里跑，所以还是要先装一下docker环境  
安装docker及docker-compose
```
apt install docker.io
apt install docker-compose
```
由于国内直接镜像比较慢，所以建议使用Docker镜像代理，在阿里云里使用”容器镜像服务->镜像加速器“，找到自己的镜像加速地址，根据不同的系统配置镜像加速器，ubuntu系统配置如下：
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://youkey.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```  
**1.3、安装Golang**  
1、下载安装包到服务器上  
```
wget https://dl.google.com/go/go1.14.1.linux-amd64.tar.gz
```
2、解压安装包  
```
tar -C /usr/local -xzf go1.14.1.linux-amd64.tar.gz
```

3、设置环境变量  
```
vim ~/.bash_profile

export PATH=$PATH:/usr/local/go/bin
export GOROOT=/usr/local/go
export GOPATH=/data/works/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
export GOBIN=$GOROOT/bin
export GOPROXY=https://goproxy.io
```
**1.4、安装其它工具**  
安装一些系统操作的必要软件  
```
apt install curl #curl工具
apt-get install tree #树形结构查看文件
apt-get install jq
```
### 2、Fabric安装和编译

**2.1、下载源码**  
```
mkdir -p $GOPATH/src/github.com/hyperledger
cd $GOPATH/src/github.com/hyperledger
git clone https://github.com/hyperledger/fabric
```
**2.2、安装相关依赖软件**
```
go get github.com/golang/protobuf/protoc-gen-go
mkdir -p $GOPATH/src/github.com/hyperledger/fabric/build/docker/gotools/bin
cp  $GOPATH/bin/protoc-gen-go $GOPATH/src/github.com/hyperledger/fabric/build/docker/gotools/bin
```
**2.3、编译Fabric模块**  
进入到Fabirc源码所在的文件夹，执行以下命令可以一次完成Fabric 5个主要模块的编译过程，具体的命令如下所示：
```
cd $GOPATH/src/github.com/hyperledger/fabric
make release  #编译fabric模块
make docker #做成镜像，如果需要
```
对于Macos系统，在编译之前需要进行以下设置：  

* 打开文件$GOPATH/src/github.com/hyperledger/fabric/Makefile
* 找到其中的第一个GO_LDFLAGS字符串的位置，在该字符串所在行的在行的末尾加上字符串-s
* 保存文件Makefile
上述命令执行完成之后，会自动将将编译好的二进制文件存放在以下路径中：

Ubuntu和Centos系统的存放路径
```
$GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin
```
Macos系统的存放路径
```
$GOPATH/src/github.com/hyperledger/fabric/release/darwin-amd64/bin
```
**2.4、Fabric模块安装**  
编译完成之后，这些模块已经已经可以被运行了，但是目前只能在编译文件所在的文件夹中运行这些模块，这样是非常不方便的。为了更加方便的使用这些模块，可以通过下面的命令将这些模块的可执行文件复制到系统目录中，这样在系统中的任何路径下面都运行这些可执行这些模块。

Ubuntu和Centos将Fabric模块编译后的文件复制到系统文件夹中的方法如下：
```
cp $GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin/* /usr/local/bin
```
Macos上面将Fabric模块编译后的文件复制到系统文件夹中的方法如下：
```
cp $GOPATH/src/github.com/hyperledger/fabric/release/darwin-amd64/bin/* /usr/local/bin
```
复制成功之后通过以下命令修改文件的执行权限，否则无法执行。
```
sudo chmod -R 775  /usr/local/bin/configtxgen
sudo chmod -R 775  /usr/local/bin/configtxlator
sudo chmod -R 775  /usr/local/bin/cryptogen
sudo chmod -R 775  /usr/local/bin/peer
sudo chmod -R 775  /usr/local/bin/orderer
```
通过上面这些命令之后，可以在系统的任何路径下面运行这些模块了。下面通过一组命令来进检查安装过程是否成功。
**2.5、检查各模块是否安装正确**  
采用 version 命令行选项

| 模块名称 | 功能 |
| --- | --- |
| peer | 主节点模块，负责存储区块链数据，运行维护链码 |
| Orderer | 交易打包、排序模块 |
| cryptogen | 组织和证书生成模块 |
| configtxgen | 区块和交易生成模块 |
| configtxlator | 区块和交易解析模块 |  

<br/>

###3、启动Fabric网络
**3.1、生成Fabric需要的证书文件**  
本例中我们将配置文件和生成的证书文件放在文件夹/opt/hyperledger/fabricconfig中。

创建存放证书的文件夹的命令如下所示：
```
mkdir -p /opt/hyperledger/fabricconfig
```
cryptogen提供了一个命令可以获取cryptogen模块所需要的配置文件的样式，该命令如下所示：
```
cryptogen showtemplate
```
可以把上述命令生成的内容复制到一个文件中稍加修改即可使用。本例中我们所使用的配置文件的内容如下所示：
```
OrdererOrgs:
  - Name: Orderer
    Domain: xuyao.com
    Specs:
      - Hostname: orderer
PeerOrgs:
  - Name: Org1
    Domain: org1.xuyao.com
    Template:
      Count: 2
    Users:
      Count: 3
  - Name: Org2
    Domain: org2.xuyao.com
    Template:
      Count: 2
    Users:
      Count: 2
```
将上述文件的内容保存到文件夹/opt/hyperledger/fabricconfig中，配置文件夹命名为：crypto-config.yaml。保存之后执行如下命令：
```
cd /opt/hyperledger/fabricconfig
cryptogen generate --config=crypto-config.yaml --output ./crypto-config
```
该命令执行完成之后我们会发现在文件夹/opt/hyperledger/fabricconfig中会新增加一个文件夹crypto-config，里面存放有本例的相关配置文件，可以通过tree命令查看生成证书文件的内容。
```
cd /opt/hyperledger/fabricconfig
tree -L 5
```
执行命令 tree -L 5 显示如下：
```
├── crypto-config
│   ├── ordererOrganizations
│   │   └── xuyao.com
│   │       ├── ca
│   │       ├── msp
│   │       ├── orderers
│   │       ├── tlsca
│   │       └── users
│   └── peerOrganizations
│       ├── org1.xuyao.com
│       │   ├── ca
│       │   ├── msp
│       │   ├── peers
│       │   ├── tlsca
│       │   └── users
│       ├── org2.xuyao.com
│       │   ├── ca
│       │   ├── msp
│       │   ├── peers
│       │   ├── tlsca
│       │   └── users
```
在上述命令显示的内容中提取出后缀为 qklszzn.com域名。本例中提取信息如下：
```
orderer.xuyao.com
peer0.org1.xuyao.com
peer1.org1.xuyao.com
peer3.org1.xuyao.com
peer0.org2.xuyao.com
peer1.org2.xuyao.com
```
通过上述步骤所有的证书文件都已经生成完毕，现在需要将测试域名映射到本机的IP地址上面，否则后面的操作可能会出现错误。
打开端映射文件,在打开的文件中设置如下内容
```
vi /etc/hosts
```
```
192.168.23.212 orderer.xuyao.com
192.168.23.212 peer0.org1.xuyao.com
192.168.23.212 peer1.org1.xuyao.com
192.168.23.212 peer3.org1.xuyao.com
192.168.23.212 peer0.org2.xuyao.com
192.168.23.212 peer1.org2.xuyao.com
```
**3.2、生成创世块**  
**3.2.1、系统创始块的生成**  
Fabric是基于区块链的分布式账本，每个账本都拥有自己的区块链，账本的区块链中会存储账本的交易，账本区块链中的第一个区块是个例外，该区块不存在交易数据而是存储配置信息，通常将账本的第一个区块成为创始块。综上所述，Fabric中账本的第一个区块是需要手动生成的。configtxgen模块是专门负责生成系统的创始块和Channel的创始块。configtxgen模块也需要一个配置文件来定义相关的属性。

在Fabric源码中提供的configtxgen模块所需要的配置文件的例子。该文件的路径是$GOPATH/src/github.com/hyperledger/fabric/sampleconfig，在这个目录下面有一个名为configtx.yaml的文件，对这个文件进行修即可使用。由于创始块文件是提供给Orderer节点使用，因此我们创建文件一个文件夹来存在Orderer节点相关的文件。文件夹创建之后把样例配置文件复制到该文件夹中。

创建存放configtxgen模块相关配置文件的文件夹的命令如下所示：
```
mkdir -p /opt/hyperledger/order/
cp -r $GOPATH/src/github.com/hyperledger/fabric/sampleconfig/configtx.yaml   /opt/hyperledger/order
cd /opt/hyperledger/order
```
对configtx.yaml进行修改，修改后的内容如下所示：
```

Profiles:

    TestTwoOrgsOrdererGenesis:
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2

    TestTwoOrgsChannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2

Organizations:
    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: /opt/hyperledger/fabricconfig/crypto-config/ordererOrganizations/xuyao.com/msp

    - &Org1

        Name: Org1MSP
        ID: Org1MSP
        MSPDir: /opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/msp
        AnchorPeers:
            - Host: peer0.org1.xuyao.com
              Port: 7051

    - &Org2
        Name: Org2MSP
        ID: Org2MSP
        MSPDir: /opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org2.xuyao.com/msp
        AnchorPeers:
            - Host: peer0.org2.xuyao.com
              Port: 7051

Orderer: &OrdererDefaults

    OrdererType: solo
    Addresses:
        - orderer.xuyao.com:7050
    BatchTimeout: 2s

    BatchSize:
        MaxMessageCount: 10
        AbsoluteMaxBytes: 98 MB
        PreferredMaxBytes: 512 KB

    Kafka:
        Brokers:
            - 127.0.0.1:9092
    Organizations:

Application: &ApplicationDefaults

    Organizations:
```
配置文件修改完成之后执行如下面命令生成创始块文件。
```
cd /opt/hyperledger/order
configtxgen -profile  TestTwoOrgsOrdererGenesis  -outputBlock  ./genesis.block
```
上述命令执行完成之后会在文件夹/opt/hyperledger/order中生成文件genesis.block。这是Fabric系统的创始块文件。  
<br/>
**3.2.2、账本创始块的生成**  
创建Channel也是通过configtxgen模块完成的，创建Channel初始块的配置文件和创建系统初始块的配置文件是一样的，具体在本例中，Channel的创始块的配置信息已经定义在本节第一部分系统创始块的生成中生成的配置文件configtx.yaml中。

创建Channel的命令如下：
```
configtxgen -profile  TestTwoOrgsChannel  -outputCreateChannelTx  ./roberttestchannel.tx -channelID  roberttestchannel
```
上述命令执行完成之后会在目录生成文件roberttestchannel.tx，该文件用来生成Channel。除此之外还需要生成相关的锚点文件，生成锚点文件需要执行以下命令：
```
configtxgen -profile  TestTwoOrgsChannel  -outputAnchorPeersUpdate ./Org1MSPanchors.tx -channelID  roberttestchannel -asOrg Org1MSP

configtxgen -profile  TestTwoOrgsChannel  -outputAnchorPeersUpdate ./Org2MSPanchors.tx -channelID  roberttestchannel -asOrg Org2MSP
```
上面命令执行完成之后会在相应的文件夹下面生成文件Org1MSPanchors.tx和Org2MSPanchors.tx，这些文件在后面会被使用到。

**3.3、Orderer节点的启动**  
Orderer节点负责交易的打包和区块的生成。Orderer节点的配置信息通常放在环境变量或者配置文件中，本例中的配置信息统一存放在配置文件中。在Fabric源码提供了Orderer启动所用到的配置文件的实例，将示例配置文件复制到Orderer的文件夹下面修改即可使用。

复制配置文件到Orderer文件夹的命令如下所示：
```
cd /opt/hyperledger/peer
cp $GOPATH/src/github.com/hyperledger/fabric/sampleconfig/orderer.yaml /opt/hyperledger/orderer
```
在模板配置文件上稍加修改即可满足本例使用，由于篇幅本例中我们只列出需要修改的部分。修改后配置文件中发生变化的内容如下：
```

General:

    LedgerType: file
    ListenAddress: 0.0.0.0
    ListenPort: 7050
    TLS:
        Enabled: false
        PrivateKey: /opt/hyperledger/fabricconfig/crypto-config/ordererOrganizations/xuyao.com/orderers/orderer.xuyao.com/tls/server.key
        Certificate: /opt/hyperledger/fabricconfig/crypto-config/ordererOrganizations/xuyao.com/orderers/orderer.xuyao.com/tls/server.crt
        RootCAs:
          - /opt/hyperledger/fabricconfig/crypto-config/ordererOrganizations/xuyao.com/orderers/orderer.xuyao.com/tls/ca.crt
        ClientAuthEnabled: false
        ClientRootCAs:
    LogLevel: debug
    LogFormat: '%{color}%{time:2006-01-02 15:04:05.000 MST} [%{module}] %{shortfunc} -> %{level:.4s} %{id:03x}%{color:reset} %{message}'
    GenesisMethod: file
    GenesisProfile: TestOrgsOrdererGenesis
    GenesisFile: /opt/hyperledger/order/orderer.genesis.block
    LocalMSPDir: /opt/hyperledger/fabricconfig/crypto-config/ordererOrganizations/xuyao.com/orderers/orderer.xuyao.com/msp
    LocalMSPID: OrdererMSP
    Profile:
        Enabled: false
        Address: 0.0.0.0:6060
    BCCSP:

        Default: SW
        SW:
            Hash: SHA2
            Security: 256
            FileKeyStore:
                KeyStore:
FileLedger:
    Location: /opt/hyperledger/order/production/orderer
    Prefix: hyperledger-fabric-ordererledger
RAMLedger:
    HistorySize: 1000
Debug:
    BroadcastTraceDir:
    DeliverTraceDir:
```
在配置文件orderer.yaml所在的目录执行如下命令启动orderer
```
orderer start
```

**3.4、Peer节点启动**  
Peer模块是Fabric的核心节点，所有的交易数据经过Orderer排序打包之后由Peer模块存储在区块链中。所有的Chaincode也是有Peer模块打包并且激活的。Peer模块的配置信息同样由环境变量和配置文件组成，本例中我们采用配置文的方式来配置peer节点的参数。在设定配置文件之前需要创建一个文件夹存放Peer模块的配置文件和区块数据。在Fabric源码中同样提供了Peer模块配置文件的示例，将示例配置文件复制到Peer模块的文件夹下面修改即可使用。

创建存储Peer模块的配置文件和区块数据的文件夹，并复制示例配置文件的命令如下所示：
```
mkdir -p /opt/hyperledger/peer
cd /opt/hyperledger/peer
cp  $GOPATH/src/github.com/hyperledger/fabric/sampleconfig/core.yaml /opt/hyperledger/peer
```
在模板配置文件上稍加修改即可使用，由于篇幅本例中我们只列出需要修改的部分。修改后Peer模块配置文件中变化的内容如下所示：
```
logging:
    peer:       debug
    cauthdsl:   warning
    gossip:     warning
    ledger:     info
    msp:        warning
    policies:   warning
    grpc:       error
    format: '%{color}%{time:2006-01-02 15:04:05.000 MST} [%{module}] %{shortfunc} -> %{level:.4s} %{id:03x}%{color:reset} %{message}'
peer:

    id: peer0.org1.qklszzn.com
    networkId: dev
    listenAddress: 0.0.0.0:7051
    chaincodeListenAddress: 0.0.0.0:7052
    address: peer0.org1.xuyao.com:7051
    addressAutoDetect: false
    gomaxprocs: -1
    gossip:

        bootstrap: 127.0.0.1:7051
        useLeaderElection: true
        orgLeader: false
        endpoint:
        maxBlockCountToStore: 100
        maxPropagationBurstLatency: 10ms
        maxPropagationBurstSize: 10
        propagateIterations: 1
        propagatePeerNum: 3
        pullInterval: 4s
        pullPeerNum: 3
        requestStateInfoInterval: 4s
        publishStateInfoInterval: 4s
        stateInfoRetentionInterval:
        publishCertPeriod: 10s
        skipBlockVerification: false
        dialTimeout: 3s
        connTimeout: 2s
        recvBuffSize: 20
        sendBuffSize: 200
        digestWaitTime: 1s
        requestWaitTime: 1s
        responseWaitTime: 2s
        aliveTimeInterval: 5s
        aliveExpirationTimeout: 25s
        reconnectInterval: 2
        externalEndpoint: peer0.org1.qklszzn.com:7051
        election:
            startupGracePeriod: 15s
            membershipSampleInterval: 1s
            leaderAliveThreshold: 10s
            leaderElectionDuration: 5s
        pvtData:
            maxPeers: 3
            minAck:   3
    events:
        address: 0.0.0.0:7053
        buffersize: 100
        timeout: 10ms
    tls:
        enabled: false
        cert:
            file: /opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/peers/peer0.org1.xuyao.com/tls/server.crt
        key:
            file: /opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/peers/peer0.org1.xuyao.com/tls/server.key
        rootcert:
            file: /opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/peers/peer0.org1.xuyao.com/tls/ca.crt
        serverhostoverride:
    fileSystemPath: /opt/hyperledger/peer/production
    BCCSP:
        Default: SW
        SW:
            Hash: SHA2
            Security: 256
            FileKeyStore:
                KeyStore:

    mspConfigPath: /opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/peers/peer0.org1.xuyao.com/msp

    localMspId: Org1MSP
    profile:
        enabled:     false
        listenAddress: 0.0.0.0:6060
    handlers:
        authFilter: "DefaultAuth"
        decorator: "DefaultDecorator
vm:
    endpoint: unix:///var/run/docker.soc
    docker:
        tls:
            enabled: false
            ca:
                file: docker/ca.crt
            cert:
                file: docker/tls.crt
            key:
                file: docker/tls.key

        attachStdout: false
        hostConfig:
            NetworkMode: host
            Dns:
            LogConfig:
                Type: json-file
                Config:
                    max-size: "50m"
                    max-file: "5"
            Memory: 2147483648
chaincode:
    peerAddress:
    id:
        path:
        name:
    builder: $(DOCKER_NS)/fabric-ccenv:$(ARCH)-$(PROJECT_VERSION)
    golang:
        runtime: $(BASE_DOCKER_NS)/fabric-baseos:$(ARCH)-$(BASE_VERSION)
    car:
        runtime: $(BASE_DOCKER_NS)/fabric-baseos:$(ARCH)-$(BASE_VERSION)
    java:
        Dockerfile:  |
            from $(DOCKER_NS)/fabric-javaenv:$(ARCH)-$(PROJECT_VERSION)
    node:
        runtime: $(BASE_DOCKER_NS)/fabric-baseimage:$(ARCH)-$(BASE_VERSION)
    startuptimeout: 300s
    executetimeout: 30s
    mode: dev
    keepalive: 0
    system:
        cscc: enable
        lscc: enable
        escc: enable
        vscc: enable
        qscc: enable
        rscc: disable
    logging:
      level:  info
      shim:   warning
      format: '%{color}%{time:2006-01-02 15:04:05.000 MST} [%{module}] %{shortfunc} -> %{level:.4s} %{id:03x}%{color:reset} %{message}'
ledger:

  blockchain:
  state:
    stateDatabase: goleveldb
    couchDBConfig:
       couchDBAddress: 127.0.0.1:5984
       username:
       password:
       maxRetries: 3
       maxRetriesOnStartup: 10
       requestTimeout: 35s
       queryLimit: 10000
  history:
    enableHistoryDatabase: true
```
在配置文件core.yaml所在的文件夹中执行以下命令启动order节点
```
export set FABRIC_CFG_PATH=/opt/hyperledger/peer
peer node start 
#或者后台启动
peer node start >> log_peer.log 2>&1 &
```

**3.5、创建通道**  
现在我们可以创建通道，创建通道的过程一共需要三个步骤。

第一步： 创建通道
```
export set CORE_PEER_LOCALMSPID=Org1MSP
export set CORE_PEER_MSPCONFIGPATH=/opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.qklszzn.com/users/Admin@org1.xuyao.com/msp

cd /opt/hyperledger/order

peer channel create -t 50s -o orderer.xuyao.com:7050 -c mychannel -f /opt/hyperledger/order/mychannel.tx
```
创建通道完成之后，会在执行命令的当前目录生成名为mychannel.block的通道创始块文件

第二步：让已经运行的Peer模块加入通道
```
export set CORE_PEER_LOCALMSPID=Org1MSP
export set CORE_PEER_ADDRESS=peer0.org1.xuyao.com:7051
export set CORE_PEER_MSPCONFIGPATH=/opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/users/Admin@org1.xuyao.com/msp

peer channel join -b /opt/hyperledger/orderer/mychannel.block
```
在上述创建通道的命令中-b后面的参数为第一步中生成的文件roberttestchannel.block，需要注意这个文件的路径。

第三步：更新锚节点
```
export set CORE_PEER_LOCALMSPID=Org1MSP
export set CORE_PEER_ADDRESS=peer0.org1.xuyao.com:7051
export set CORE_PEER_MSPCONFIGPATH=/opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/users/Admin@org1.xuyao.com/msp


peer channel update -o orderer.xuyao.com:7050 -c mychannel -f  /opt/hyperledger/order/Org1MSPanchors.tx
```

**3.6、Chaincode的部署和调用**  
现在可以部署一个Chaincode（关于Chaincode的详细内容在本书的第七章会有详细的介绍）来测试Peer节点和Orderer节点的部署是否正确。这里采用Fabric源码自带的例子来作为测试Chaincode。测试用Chaincode的源代码路径如下所示
```
$GOPATH/src/github.com/hyperledger/fabric/examples/chaincode/go/example02/cmd
```
Chaincode相关的测试一共有四个步骤。
第一步： 部署chaincode代码
```
export set CORE_PEER_LOCALMSPID=Org1MSP
export set CORE_PEER_ADDRESS=peer0.org1.xuyao.com:7051
export set CORE_PEER_MSPCONFIGPATH=/opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/users/Admin@org1.xuyao.com/msp


peer chaincode install -n xuyao_test_cc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/example02/cmd
```
第二步： 实例化chaincode代码
```
export set CORE_PEER_LOCALMSPID=Org1MSP
export set CORE_PEER_ADDRESS=peer0.org1.xuyao.com:7051
export set CORE_PEER_MSPCONFIGPATH=/opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/users/Admin@org1.xuyao.com/msp

peer chaincode instantiate -o  orderer.xuyao.com:7050 -C mychannel -n xuyao_test_cc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR  ('Org1MSP.member','Org2MSP.member')"
```
第三步：通过chaincode写入数据
```
export set CORE_PEER_LOCALMSPID=Org1MSP
export set CORE_PEER_ADDRESS=peer0.org1.xuyao.com:7051
export set CORE_PEER_MSPCONFIGPATH=/opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/users/Admin@org1.xuyao.com/msp

peer chaincode invoke -o orderer.xuyao.com:7050 -C mychannel -n xuyao_test_cc -c '{"Args":["invoke","a","b","1"]}'
```
第四步：通过chaincode查询数据
```
export set CORE_PEER_LOCALMSPID=Org1MSP
export set CORE_PEER_ADDRESS=peer0.org1.xuyao.com:7051
export set CORE_PEER_MSPCONFIGPATH=/opt/hyperledger/fabricconfig/crypto-config/peerOrganizations/org1.xuyao.com/users/Admin@org1.xuyao.com/msp

peer chaincode query -C mychannel -n xuyao_test_cc -c '{"Args":["query","a"]}'
```
如果上述命令都能正确执行，那么一个简单的Fabric系统就已经部署完成了。
