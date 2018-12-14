---
layout: post
title: "Fabric多台服务器的部署(二)"
date: 2018-12-14 14:27:46 +0800
comments: true
categories: 
---
### 3、下载Fabric镜像
#####3.1.1 下载Fabric 源码
 1、在后面的例子中，有些地方需要用到源码中提到的工具来编译和生成证书等，这里需要下载一下Fabric的源码，在本地下载Fabric官方提供的源码
```
git clone git@github.com:hyperledger/fabric.git
```
需要注意的时候，需要把下载好的源码放在$GOPATH里，这个环境变量是最初安装GO的设置的，如
```
echo $GOPATH
/Users/xuyao/go
```
那么fabric的源码地址路径应该是
```
/Users/xuyao/go/src/github.com/hyperledger/fabric/
```
<!-- more -->
##### 3.1.2 下载Fabric镜像
 &emsp;如果你看过Fabric官方的快速入门文档[https://hyperledgercn.github.io/hyperledgerDocs/getting_started/](https://hyperledgercn.github.io/hyperledgerDocs/getting_started/)，应该会对Fabric的镜像文件有个了解，在fabric/examples/e2e_cli文件夹下有个download-dockerimages.sh的脚本文件，他会帮你一键下载所需的镜像文件。
```
# 使脚本可执行
chmod +x download-dockerimages.sh
# 执行脚本
./download-dockerimages.sh
```
当然也可以使用下面命令下载镜像(需要翻墙)
```
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.0 
#此处是下载fabric-samples，如果想快速使用byfn可以参考此脚本，后面1.2.0是对应的版本号
```
也可以根据自己项目需求下载不同的版本
```
curl -sSL http://bit.ly/2ysbOFE | bash -s <fabric> <fabric-ca> <thirdparth>
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.0 1.2.0 0.4.10
#后面三个参数分别代表Fabric, Fabric-ca和第三方平台镜像版本号
```
无法翻墙，可以用下面命令
```
curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh | bash -s 1.2.0 1.2.0 0.4.10
```
下载完成后使用docker images可以看到下载的镜像文件
```
docker images #查看所以镜像文件
hyperledger/fabric-ca          latest               35311d8617b4        7 days ago          240 MB
hyperledger/fabric-ca          x86_64-1.0.0-alpha   35311d8617b4        7 days ago          240 MB
hyperledger/fabric-couchdb     latest               f3ce31e25872        7 days ago          1.51 GB
hyperledger/fabric-couchdb     x86_64-1.0.0-alpha   f3ce31e25872        7 days ago          1.51 GB
hyperledger/fabric-kafka       latest               589dad0b93fc        7 days ago          1.3 GB
hyperledger/fabric-kafka       x86_64-1.0.0-alpha   589dad0b93fc        7 days ago          1.3 GB
hyperledger/fabric-zookeeper   latest               9a51f5be29c1        7 days ago          1.31 GB
hyperledger/fabric-zookeeper   x86_64-1.0.0-alpha   9a51f5be29c1        7 days ago          1.31 GB
hyperledger/fabric-orderer     latest               5685fd77ab7c        7 days ago          182 MB
hyperledger/fabric-orderer     x86_64-1.0.0-alpha   5685fd77ab7c        7 days ago          182 MB
hyperledger/fabric-peer        latest               784c5d41ac1d        7 days ago          184 MB
hyperledger/fabric-peer        x86_64-1.0.0-alpha   784c5d41ac1d        7 days ago          184 MB
hyperledger/fabric-javaenv     latest               a08f85d8f0a9        7 days ago          1.42 GB
hyperledger/fabric-javaenv     x86_64-1.0.0-alpha   a08f85d8f0a9        7 days ago          1.42 GB
hyperledger/fabric-ccenv       latest               91792014b61f        7 days ago          1.29 GB
hyperledger/fabric-ccenv       x86_64-1.0.0-alpha   91792014b61f        7 days ago          1.29 GB
```
##### 3.1.3 常用的Docker 命令
```
#停止正在运行的所有网络
docker stop -f $(docker ps -aq)

#删除正在运行的所有网络
docker rm -f $(docker ps -aq)

#删除某个镜像
docker rmi <IMAGE ID>

#查看某个容器的日志
docker logs -f  < container-id >

#进入某个容器内部
docker exec -it < container-id > bash

#查看docker 运行状态
service docker status

# 启动/关闭docker
sudo service docker start|stop
```

#### 4、下载 Fabric-sample 源码
1、fabric-sample里有很多快速启动fabric的例子，官方文档给了一个快速启动first-network的例子 [https://hyperledgercn.github.io/hyperledgerDocs/build_network_zh/](https://hyperledgercn.github.io/hyperledgerDocs/build_network_zh/)，在first-network里面，每个组织持有2个peer节点，以及一个“solo”排序服务，这里一个比较简单的fabric网络，如果只是想跑一下fabric网络，试下手感的话，可以按照官方的文档来跑一篇。  
 &emsp; 这里我们用的是官网提供的另一个例子，balance-transfer，它是一个集成了nodejs sdk的一个node.js的简单例子。
```
#下载fabric-sample
git clone git@github.com:hyperledger/fabric-samples.git
```
 2、后期我们的项目都是用nodejs来做的，所以很多基础的东西可以复用balance-transfer里的，现在我们可以自己建了项目仓库叫trace_redwine，把fabric-samples/balance-transfer/里的文件全部copy到我们的trace_redwine里,删除一些不必要的文件，这里一个自己的fabric项目雏形就出来了。
#### 5、修改配置脚本
##### 5.1.1 修改crypto-config.yaml文件
 &emsp; crypto-config.yaml文件是配置节点信息的，你可以配置你的orderer,peer数量，以及域名的设置，主要配置的是OrdererOrgs和PeerOrgs  
OrdererOrgs的配置
```
OrdererOrgs:
  - Name: Orderer
    Domain: xuyao.com
    Specs:
      - Hostname: orderer1
      - Hostname: orderer2
```
PeerOrgs的配置信息
```
PeerOrgs:
  - Name: Org1 #组织1
    Domain: org1.xuyao.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
  - Name: Org2 #组织2
    Domain: org2.xuyao.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
```
##### 5.1.2 使用generateArtifacts.sh脚本生成证书文件
&emsp; 现在我们需要生成一些证书和启动这个fabric网络，生成证书的话我们可以使用官网提供的generateArtifacts.sh脚本来生成，文件路径在/gopath/src/github.com/hyperledger/fabric/examples/e2e_cli/下，稍微修改，因为我只需要生成证书，创世纪块，和channel.tx,具体脚本如下，**需要说明的是如果你只是运行官方的first-network，可以忽略这些配置，因为first-network已经帮你生成好了这些东西**
```
#!/usr/bin/env bash
CRYPTOGEN=$PWD/bin/cryptogen
CONFIGTXGEN=$PWD/bin/configtxgen
CHANNEL_NAME=$1
: ${CHANNEL_NAME:="mychannel"}
function buildCryptogenTools(){
	if [ -f "$CONFIGTXGEN" ]; then
	    echo
	else
	    echo "configtxgen not found"
	    exit 1
	fi
}

function createPeerTemplate() {
    cp ../docker-composer-peer-template.yaml ../docker-composer
}

## Generates Org certs using cryptogen tool
function generateCerts (){
	echo
	echo "##########################################################"
	echo "##### Generate certificates using cryptogen tool #########"
	echo "##########################################################"
	if [ -d "crypto-config" ]; then
      rm -Rf crypto-config
    fi
	${CRYPTOGEN} generate --config=./crypto-config.yaml
	echo
}

## Generate orderer genesis block , channel configuration transaction and anchor peer update transactions
function generateChannelArtifacts() {
	echo "##########################################################"
	echo "#########  Generating Orderer Genesis block ##############"
	echo "##########################################################"
	${CONFIGTXGEN} -profile TwoOrgsOrdererGenesis -outputBlock ./genesis.block

	echo
	echo "#################################################################"
	echo "### Generating channel configuration transaction 'channel.tx' ###"
	echo "#################################################################"
	${CONFIGTXGEN} -profile TwoOrgsChannel -outputCreateChannelTx ./mychannel.tx -channelID ${CHANNEL_NAME} 
}

buildCryptogenTools
generateCerts
generateChannelArtifacts
```
generateCerts是生成证书的方法，它会加载crypto-config.yaml里的配置信息，并根据你的配置信息来生成相应的证书。  
generateChannelArtifacts是生成orderer的创世纪块（genesis.block）和channel.tx文件。  
上面的脚本运行之后，整个Fabric网络需要的一些证书就完成了，以本地例子为例，生成如下的一些文件数据
```
├── crypto-config
│   ├── ordererOrganizations
│   │   └── mbasechain.com
│   │       ├── ca
│   │       │   ├── 2ae1db347d3137d66fda66f59fb33cd2c3c22a4aadf7f97f800491fb91a20c32_sk
│   │       │   └── ca.mbasechain.com-cert.pem
│   │       ├── msp
│   │       │   ├── admincerts
│   │       │   │   └── Admin@mbasechain.com-cert.pem
│   │       │   ├── cacerts
│   │       │   │   └── ca.mbasechain.com-cert.pem
│   │       │   └── tlscacerts
│   │       │       └── tlsca.mbasechain.com-cert.pem
│   │       ├── orderers
│   │       │   ├── orderer.mbasechain.com
│   │       │   │   └── msp
│   │       │   │       └── keystore
│   │       │   ├── orderer1.mbasechain.com
│   │       │   │   ├── msp
│   │       │   │   │   ├── admincerts
│   │       │   │   │   │   └── Admin@mbasechain.com-cert.pem
│   │       │   │   │   ├── cacerts
│   │       │   │   │   │   └── ca.mbasechain.com-cert.pem
│   │       │   │   │   ├── keystore
│   │       │   │   │   │   └── 9af18a6c9dd8f34db73a70c1d066e6948216db534af9e4f89029158089d56cf1_sk
│   │       │   │   │   ├── signcerts
│   │       │   │   │   │   └── orderer1.mbasechain.com-cert.pem
│   │       │   │   │   └── tlscacerts
│   │       │   │   │       └── tlsca.mbasechain.com-cert.pem
│   │       │   │   └── tls
│   │       │   │       ├── ca.crt
│   │       │   │       ├── server.crt
│   │       │   │       └── server.key
│   │       │   └── orderer2.mbasechain.com
│   │       │       ├── msp
│   │       │       │   ├── admincerts
│   │       │       │   │   └── Admin@mbasechain.com-cert.pem
│   │       │       │   ├── cacerts
│   │       │       │   │   └── ca.mbasechain.com-cert.pem
│   │       │       │   ├── keystore
│   │       │       │   │   └── fb54a7774228bf31311b5aa0ab58b4c0d3f9aa0e83bb1906cb6926c033445976_sk
│   │       │       │   ├── signcerts
│   │       │       │   │   └── orderer2.mbasechain.com-cert.pem
│   │       │       │   └── tlscacerts
│   │       │       │       └── tlsca.mbasechain.com-cert.pem
│   │       │       └── tls
│   │       │           ├── ca.crt
│   │       │           ├── server.crt
│   │       │           └── server.key
│   │       ├── tlsca
│   │       │   ├── 9ceda704a49805f19c0f3cde2c1d42096d8e14da697567b0a02b051656f735de_sk
│   │       │   └── tlsca.mbasechain.com-cert.pem
│   │       └── users
│   │           └── Admin@mbasechain.com
│   │               ├── msp
│   │               │   ├── admincerts
│   │               │   │   └── Admin@mbasechain.com-cert.pem
│   │               │   ├── cacerts
│   │               │   │   └── ca.mbasechain.com-cert.pem
│   │               │   ├── keystore
│   │               │   │   └── 1c7c1a76c08e30990192ed0f6eb24c2efd7492b684f92ebf844591fb38d8b020_sk
│   │               │   ├── signcerts
│   │               │   │   └── Admin@mbasechain.com-cert.pem
│   │               │   └── tlscacerts
│   │               │       └── tlsca.mbasechain.com-cert.pem
│   │               └── tls
│   │                   ├── ca.crt
│   │                   ├── client.crt
│   │                   └── client.key
│   └── peerOrganizations
│       ├── org1.mbasechain.com
│       │   ├── ca
│       │   │   ├── 6cb226be4aec981541e6cbdd66742139c1157e88afe06edfb16f12ecc3056bf5_sk
│       │   │   └── ca.org1.mbasechain.com-cert.pem
│       │   ├── msp
│       │   │   ├── admincerts
│       │   │   │   └── Admin@org1.mbasechain.com-cert.pem
│       │   │   ├── cacerts
│       │   │   │   └── ca.org1.mbasechain.com-cert.pem
│       │   │   ├── config.yaml
│       │   │   └── tlscacerts
│       │   │       └── tlsca.org1.mbasechain.com-cert.pem
│       │   ├── peers
│       │   │   ├── peer0.org1.mbasechain.com
│       │   │   │   ├── msp
│       │   │   │   │   ├── admincerts
│       │   │   │   │   │   └── Admin@org1.mbasechain.com-cert.pem
│       │   │   │   │   ├── cacerts
│       │   │   │   │   │   └── ca.org1.mbasechain.com-cert.pem
│       │   │   │   │   ├── config.yaml
│       │   │   │   │   ├── keystore
│       │   │   │   │   │   └── b9d2875104a103f635c8ec21757710455eec0fbfb2c59f7b33a09461b4138267_sk
│       │   │   │   │   ├── signcerts
│       │   │   │   │   │   └── peer0.org1.mbasechain.com-cert.pem
│       │   │   │   │   └── tlscacerts
│       │   │   │   │       └── tlsca.org1.mbasechain.com-cert.pem
│       │   │   │   └── tls
│       │   │   │       ├── ca.crt
│       │   │   │       ├── server.crt
│       │   │   │       └── server.key
│       │   │   └── peer1.org1.mbasechain.com
│       │   │       ├── msp
│       │   │       │   ├── admincerts
│       │   │       │   │   └── Admin@org1.mbasechain.com-cert.pem
│       │   │       │   ├── cacerts
│       │   │       │   │   └── ca.org1.mbasechain.com-cert.pem
│       │   │       │   ├── config.yaml
│       │   │       │   ├── keystore
│       │   │       │   │   └── 68da53988aa0b3e50b6a90ac90495b4021fdb3456bbca375aca13cf206d18b0c_sk
│       │   │       │   ├── signcerts
│       │   │       │   │   └── peer1.org1.mbasechain.com-cert.pem
│       │   │       │   └── tlscacerts
│       │   │       │       └── tlsca.org1.mbasechain.com-cert.pem
│       │   │       └── tls
│       │   │           ├── ca.crt
│       │   │           ├── server.crt
│       │   │           └── server.key
│       │   ├── tlsca
│       │   │   ├── b2ef19d2488129396546090955ddf2d1ec267cd699dc2218516df72f5ea235f4_sk
│       │   │   └── tlsca.org1.mbasechain.com-cert.pem
│       │   └── users
│       │       ├── Admin@org1.mbasechain.com
│       │       │   ├── msp
│       │       │   │   ├── admincerts
│       │       │   │   │   └── Admin@org1.mbasechain.com-cert.pem
│       │       │   │   ├── cacerts
│       │       │   │   │   └── ca.org1.mbasechain.com-cert.pem
│       │       │   │   ├── keystore
│       │       │   │   │   └── 12f72b7332170a7b8e4214eb268c419053a2386f34b149f3e7c5820f767a2d30_sk
│       │       │   │   ├── signcerts
│       │       │   │   │   └── Admin@org1.mbasechain.com-cert.pem
│       │       │   │   └── tlscacerts
│       │       │   │       └── tlsca.org1.mbasechain.com-cert.pem
│       │       │   └── tls
│       │       │       ├── ca.crt
│       │       │       ├── client.crt
│       │       │       └── client.key
│       │       └── User1@org1.mbasechain.com
│       │           ├── msp
│       │           │   ├── admincerts
│       │           │   │   └── User1@org1.mbasechain.com-cert.pem
│       │           │   ├── cacerts
│       │           │   │   └── ca.org1.mbasechain.com-cert.pem
│       │           │   ├── keystore
│       │           │   │   └── 610899489aa3d881dcc6e9f3d421981003b1504e5b15c00426edd69505ef3519_sk
│       │           │   ├── signcerts
│       │           │   │   └── User1@org1.mbasechain.com-cert.pem
│       │           │   └── tlscacerts
│       │           │       └── tlsca.org1.mbasechain.com-cert.pem
│       │           └── tls
│       │               ├── ca.crt
│       │               ├── client.crt
│       │               └── client.key
│       ├── org2.mbasechain.com
│       │   ├── ca
│       │   │   ├── 018041cad6bac4a43ab80e736ba333a759a14ee6e88226944ec904dfcba633ab_sk
│       │   │   └── ca.org2.mbasechain.com-cert.pem
│       │   ├── msp
│       │   │   ├── admincerts
│       │   │   │   └── Admin@org2.mbasechain.com-cert.pem
│       │   │   ├── cacerts
│       │   │   │   └── ca.org2.mbasechain.com-cert.pem
│       │   │   ├── config.yaml
│       │   │   └── tlscacerts
│       │   │       └── tlsca.org2.mbasechain.com-cert.pem
│       │   ├── peers
│       │   │   ├── peer0.org2.mbasechain.com
│       │   │   │   ├── msp
│       │   │   │   │   ├── admincerts
│       │   │   │   │   │   └── Admin@org2.mbasechain.com-cert.pem
│       │   │   │   │   ├── cacerts
│       │   │   │   │   │   └── ca.org2.mbasechain.com-cert.pem
│       │   │   │   │   ├── config.yaml
│       │   │   │   │   ├── keystore
│       │   │   │   │   │   └── cfb79d8ab3c543a8732227cd2b441ce30e82ff1d4aad5f59c1338454edeeddf1_sk
│       │   │   │   │   ├── signcerts
│       │   │   │   │   │   └── peer0.org2.mbasechain.com-cert.pem
│       │   │   │   │   └── tlscacerts
│       │   │   │   │       └── tlsca.org2.mbasechain.com-cert.pem
│       │   │   │   └── tls
│       │   │   │       ├── ca.crt
│       │   │   │       ├── server.crt
│       │   │   │       └── server.key
│       │   │   └── peer1.org2.mbasechain.com
│       │   │       ├── msp
│       │   │       │   ├── admincerts
│       │   │       │   │   └── Admin@org2.mbasechain.com-cert.pem
│       │   │       │   ├── cacerts
│       │   │       │   │   └── ca.org2.mbasechain.com-cert.pem
│       │   │       │   ├── config.yaml
│       │   │       │   ├── keystore
│       │   │       │   │   └── b92cbb64895e522299770c9f84ad9954753f348d926da956d4458cf545b99ef6_sk
│       │   │       │   ├── signcerts
│       │   │       │   │   └── peer1.org2.mbasechain.com-cert.pem
│       │   │       │   └── tlscacerts
│       │   │       │       └── tlsca.org2.mbasechain.com-cert.pem
│       │   │       └── tls
│       │   │           ├── ca.crt
│       │   │           ├── server.crt
│       │   │           └── server.key
│       │   ├── tlsca
│       │   │   ├── 2674e511ae5eb66b4964a2a93cc917f5e8fbd49e95fc97d12b546c9ce933e932_sk
│       │   │   └── tlsca.org2.mbasechain.com-cert.pem
│       │   └── users
│       │       ├── Admin@org2.mbasechain.com
│       │       │   ├── msp
│       │       │   │   ├── admincerts
│       │       │   │   │   └── Admin@org2.mbasechain.com-cert.pem
│       │       │   │   ├── cacerts
│       │       │   │   │   └── ca.org2.mbasechain.com-cert.pem
│       │       │   │   ├── keystore
│       │       │   │   │   └── 4ca087cf7e8ecd260b84782ec8456b693ada673b902d525fb0dfcbf4bc797e38_sk
│       │       │   │   ├── signcerts
│       │       │   │   │   └── Admin@org2.mbasechain.com-cert.pem
│       │       │   │   └── tlscacerts
│       │       │   │       └── tlsca.org2.mbasechain.com-cert.pem
│       │       │   └── tls
│       │       │       ├── ca.crt
│       │       │       ├── client.crt
│       │       │       └── client.key
│       │       └── User1@org2.mbasechain.com
│       │           ├── msp
│       │           │   ├── admincerts
│       │           │   │   └── User1@org2.mbasechain.com-cert.pem
│       │           │   ├── cacerts
│       │           │   │   └── ca.org2.mbasechain.com-cert.pem
│       │           │   ├── keystore
│       │           │   │   └── 950ff48168df356652a4fa6d5f9d3aa7d6d7694045cc3bb7f83587686794a345_sk
│       │           │   ├── signcerts
│       │           │   │   └── User1@org2.mbasechain.com-cert.pem
│       │           │   └── tlscacerts
│       │           │       └── tlsca.org2.mbasechain.com-cert.pem
│       │           └── tls
│       │               ├── ca.crt
│       │               ├── client.crt
│       │               └── client.key
├── crypto-config.yaml
├── genesis.block
├── mychannel.tx
```
##### 未完待续。。。


##### 参考资料
https://hyperledgercn.github.io/hyperledgerDocs/getting_started/  
https://www.codetd.com/article/2709144  
https://hyperledgercn.github.io/hyperledgerDocs/build_network_zh/