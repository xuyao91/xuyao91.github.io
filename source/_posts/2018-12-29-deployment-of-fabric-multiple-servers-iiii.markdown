---
layout: post
title: "Fabric多台服务器的部署(四)"
date: 2018-12-29 16:51:08 +0800
comments: true
categories: 
---
#### 7、创建channel和chaincode
 &emsp; 之前我们说过，本项目是一个nodejs的项目，是根据[https://github.com/hyperledger/fabric-samples](https://github.com/hyperledger/fabric-samples)里的balance_transfer稍作修改来的，所以这里我们创建channel和chaincode，包括数据上链都会用nodejs sdk来做。
###### 7.1、启动nodejs服务
 &emsp;进入我们的trace_wine项目，使用npm安装项目所需的package,
```
npm install
```
安装完后，我们现在要启动node服务，我们可以看下，当前目录下面的个runApp.sh的脚本文件，看下里面内容就知道，这是个启动node服务的脚本
<!-- more -->
![项目的基本文件](https://upload-images.jianshu.io/upload_images/1796624-895fa4b8dae442f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
现在我们来运行一下这个脚本文件，使用命令
```
./runApp.sh
```
查看是否启动成功，进入项目下的logs文件夹，找到当前的日志文件，打开看一下
```
tail -f 2018-11-28.log

[2018-11-28 15:06:48.699] [INFO] Helper - Successfully loaded member from persistence
[2018-11-28 15:07:15.485] [INFO] app - ****************** SERVER STARTED ************************
[2018-11-28 15:07:15.488] [INFO] app - ***************  http://localhost:4000  ******************
```
可以看到node服务已经启动成功，且端口号为4000，具体的文档可查看官方的[readme](https://github.com/hyperledger/fabric-samples/blob/release-1.3/balance-transfer/README.md)

###### 7.2、创建Ca用户
 &emsp;Fabric CA为每个上链、查询者提供了注册用户，生成用户证书(ECerts)的功能，我们现在可以通过REST APIs来与ca server交互，我们先看下node里注册用户，生成证书的代码
```
///////////////////////////////////////////////////////////////////////////////
///////////////////////// REST ENDPOINTS START HERE ///////////////////////////
///////////////////////////////////////////////////////////////////////////////
// Register and enroll user
app.post('/users', async function(req, res) {
	var username = req.body.username;
	var orgName = req.body.orgName;
	logger.debug('End point : /users');
	logger.debug('User name : ' + username);
	logger.debug('Org name  : ' + orgName);
	if (!username) {
		res.json(getErrorMessage('\'username\''));
		return;
	}
	if (!orgName) {
		res.json(getErrorMessage('\'orgName\''));
		return;
	}
	var token = jwt.sign({
		exp: Math.floor(Date.now() / 1000) + parseInt(hfc.getConfigSetting('jwt_expiretime')),
		username: username,
		orgName: orgName
	}, app.get('secret'));
	let response = await helper.getRegisteredUser(username, orgName, true);
	logger.debug('-- returned from registering the username %s for organization %s',username,orgName);
	if (response && typeof response !== 'string') {
		logger.debug('Successfully registered the username %s for organization %s',username,orgName);
		response.token = token;
		res.json(response);
	} else {
		logger.debug('Failed to register the username %s for organization %s with::%s',username,orgName,response);
		res.json({success: false, message: response});
	}

});
```
可以看到当用户传进来username和orgName时，node首先会使用jwt将他们生成一个Token，然后通过getRegisteredUser()方法生成用户的证书，最终将jwt生成的token返回给用户。  
 &emsp;下面我们来调用一个这个接口
```
#在Org1上注册和生成一个新用户，用户名为Jim
curl -s -X POST http://localhost:4000/users -H "content-type: application/x-www-form-urlencoded" -d 'username=Jim&orgName=Org1'
```
```
#返回结果
{
  "success": true,
  "secret": "RaxhMgevgJcm",
  "message": "Jim enrolled Successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NDQ1MjIxMTgsInVzZXJuYW1lIjoiZG9jIiwib3JnTmFtZSI6Ik9yZzEiLCJpYXQiOjE1NDQ0MzU3MTh9.X8DuFxUSmsRTe7v7iMft8A7LxzpvGyhnufBLQTZ3F8I"
}
```
上面的token就是我们想要的，之后每次请求接口都会用token来验证用户信息,更多的jwt信息，可以查看这里[https://jwt.io/](https://jwt.io/)，另外用户生成的证书保存在服务器端，是标准的X.509证书格式，可以打开看一下这个证书
```
{"name":"Jim","mspid":"Org1MSP","roles":null,"affiliation":"","enrollmentSecret":"",
"enrollment":{"signingIdentity":"72d623e2104955b39ca6b6383a278b4b0a253e45a83b26044edf193563655367","
identity":{"certificate":"
-----BEGIN CERTIFICATE-----
\nMIICkDCCAjegAwIBAgIUX+durkyChVVZ5LNZekudT17CcJUwCgYIKoZIzj0EAwIw\n
eTELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh\n
biBGcmFuY2lzY28xHDAaBgNVBAoTE29yZzEubWJhc2VjaGFpbi5jb20xHzAdBgNV\n
BAMTFmNhLm9yZzEubWJhc2VjaGFpbi5jb20wHhcNMTgxMTIzMDk0NjAwWhcNMTkx\n
MTIzMDk1MTAwWjBAMTAwDQYDVQQLEwZjbGllbnQwCwYDVQQLEwRvcmcxMBIGA1UE\n
CxMLZGVwYXJ0bWVudDExDDAKBgNVBAMTA2RvYzBZMBMGByqGSM49AgEGCCqGSM49\n
AwEHA0IABGUZz9n5KvXF9H0l4HSdCvegNE2A0pkk2w1oQLcJW9FaZvr7rBDG3/bl\nauV26OoHJX7Yo/N
x2JyQrQzYRoPKfOWjgdUwgdIwDgYDVR0PAQH/BAQDAgeAMAwG\n
A1UdEwEB/wQCMAAwHQYDVR0OBBYEFCGGAIKaIFfBpkYuR2rjxvGle+uRMCsGA1Ud\n
IwQkMCKAIGyyJr5K7JgVQebL3WZ0ITnBFX6Ir+Bu37FvEuzDBWv1MGYGCCoDBAUG\n
BwgBBFp7ImF0dHJzIjp7ImhmLkFmZmlsaWF0aW9uIjoib3JnMS5kZXBhcnRtZW50\n
MSIsImhmLkVucm9sbG1lbnRJRCI6ImRvYyIsImhmLlR5cGUiOiJjbGllbnQifX0w\n
CgYIKoZIzj0EAwIDRwAwRAIgYnTD6Pkx1+R4R77TztW3/oQb1h+5/3ELYtAsIuz7\n
D8wCIBiOmE/uySQvkgzUcCsVRtaUMg0M9zioKBYHiPUxeNJo\n
-----END CERTIFICATE-----\n"}}}%
```
###### 7.2、创建Channel
 &emsp; 同样我们使用node服务提供的接口来创建Channel通道，具体创建channel的js代码可以看[这里](https://github.com/hyperledger/fabric-samples/blob/release-1.3/balance-transfer/app.js#L142)
```
curl -s -X POST \
  http://localhost:4000/channels \
  -H "authorization: Bearer Token" \
  -H "content-type: application/json" \
  -d '{
	"channelName":"mychannel",
	"channelConfigPath":"../artifacts/channel/mychannel.tx"
}'
```
authorization里的Token是上面我们得到的token， channelName是我们要创建通道的名字，channelConfigPath指定mychannel.tx文件路径，创建成功，返回如下结果
```
{"success":true,"message":"Channel 'mychannel' created Successfully"}
```
###### 7.3、将Channel加入到Org里
 &emsp;现在我们需要将channel加入到Org里，这样Org就可以使用这个channel，同样是通过node接口来实现
```
curl -s -X POST \
  http://localhost:4000/channels/mychannel/peers \
  -H "authorization: Bearer Token" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org1.xuyao.com","peer1.org1.xuyao.com"]
}'
```
返回结果
```
{"success":true,"message":"Successfully joined peers in organization Org1 to the channel:mychannel"}
```
###### 7.4、编写和安装智能合约(chaincode)
###### 7.4.1、编写智能合约(chaincode)
 &emsp;channel安装好后，我们需要编写智能合约，这里我贴一个我用go写的一个chaincode,功能很简单就是把数据上链，查询
```
package main

import (
	"bytes"
	"fmt"

	"github.com/hyperledger/fabric/core/chaincode/shim"
	pb "github.com/hyperledger/fabric/protos/peer"
)

//Chaincode implementation
type SimpleChaincode struct {
}


// ===================================================================================
// Main
// ===================================================================================
func main() {
	err := shim.Start(new(SimpleChaincode))
	if err != nil {
		fmt.Printf("Error starting Simple chaincode: %s", err)
	}
}

// Init initializes chaincode
// ===========================
func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
	return shim.Success(nil)
}

// Invoke - Our entry point for Invocations
// ========================================
func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
	function, args := stub.GetFunctionAndParameters()
	fmt.Printf("function: %s,args: %s\n", function, args)

	// Handle different functions
	if function == "save" { //create a new marble
		return t.save(stub, args)
	} else if function == "query" { //find Data based on an ad hoc rich query
		return t.query(stub, args)
	}

	fmt.Println("invoke did not find func: " + function) //error
	return shim.Error("Received unknown function invocation")
}

// ============================================================
// - create a new data, store into chaincode state
// ============================================================
func (t *SimpleChaincode) save(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	
	if len(args) != 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	err := stub.PutState(args[0], []byte(args[1]))
	if err != nil {
		return shim.Error(err.Error())
	}
	fmt.Println("- end save business")
	return shim.Success(nil)
	

}

// ===== Add hoc rich query ========================================================
// query method uses a query string to perform a query for data.
// Query string matching state database syntax is passed in and executed as is.
// Supports ad hoc queries that can be defined at runtime by the client.
// Only available on state databases that support rich query (e.g. CouchDB)
// =========================================================================================
func (t *SimpleChaincode) query(stub shim.ChaincodeStubInterface, args []string) pb.Response {

	//   0
	// "queryString"
	if len(args) < 1 {
		return shim.Error("Incorrect number of arguments. Expecting 1")
	}

	queryString := args[0]

	queryResults, err := getQueryResultForQueryString(stub, queryString)
	if err != nil {
		return shim.Error(err.Error())
	}
	return shim.Success(queryResults)
}

// =========================================================================================
// getQueryResultForQueryString executes the passed in query string.
// Result set is built and returned as a byte array containing the JSON results.
// =========================================================================================
func getQueryResultForQueryString(stub shim.ChaincodeStubInterface, queryString string) ([]byte, error) {

	fmt.Printf("- getQueryResultForQueryString queryString:\n%s\n", queryString)

	resultsIterator, err := stub.GetQueryResult(queryString)
	if err != nil {
		return nil, err
	}
	defer resultsIterator.Close()

	// buffer is a JSON array containing QueryRecords
	var buffer bytes.Buffer
	buffer.WriteString("[")

	bArrayMemberAlreadyWritten := false
	for resultsIterator.HasNext() {
		queryResponse, err := resultsIterator.Next()
		if err != nil {
			return nil, err
		}
		// Add a comma before array members, suppress it for the first array member
		if bArrayMemberAlreadyWritten == true {
			buffer.WriteString(",")
		}
		buffer.WriteString("{\"Key\":")
		buffer.WriteString("\"")
		buffer.WriteString(queryResponse.Key)
		buffer.WriteString("\"")

		buffer.WriteString(", \"Record\":")
		// Record is a JSON object, so we write as-is
		buffer.WriteString(string(queryResponse.Value))
		buffer.WriteString("}")
		bArrayMemberAlreadyWritten = true
	}
	buffer.WriteString("]")

	fmt.Printf("- getQueryResultForQueryString queryResult:\n%s\n", buffer.String())

	return buffer.Bytes(), nil
}
```
save()函数将传上来的数据上链，query()函数通过交易id来查询区块, getQueryResultForQueryString()函数通过交易里的字段来查询区块，目前只支持couchdb查询，leveldb不支持这样查。
###### 7.4.2、 安装chaincode
现在我们来安装chaincode
```
curl -s -X POST \
  http://localhost:4000/chaincodes \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org1.xuyao.com","peer1.org1.xuyao.com"],
	"chaincodeName":"mycc",
	"chaincodePath":"github.com/example_cc/go",
	"chaincodeType": "golang",
	"chaincodeVersion":"v0"
}'
```
chaincodeName是智能合约的名字， chaincodePath是智能合约的路径， chaincodeType是编写智能合约的语言，是golang和nodejs两种，chaincodeVersion是智能合约的版本号，值得注意的是如果是升级智能合约的话，需要加上upgrade: true的参数，而且版本号也要向上升级。这是升级的接口，如下:
```
curl -s -X POST \
  http://localhost:4000/chaincodes \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json" \
  -d '{
    "peers": ["peer0.org1.xuyao.com","peer1.org1.xuyao.com"],
    "chaincodeName":"mycc",
    "chaincodePath":"github.com/example_cc/go",
    "chaincodeType": "golang",
    "chaincodeVersion":"v1",
    "upgrade": "true"
}'
```
安装成功后，返回如下结果
```
{"success":true,"message":"Successfully install chaincode"}
```
###### 7.4.3、实例化chaincode
```
curl -s -X POST \
  http://localhost:4000/channels/mychannel/chaincodes \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json" \
  -d '{
	"chaincodeName":"mycc",
	"chaincodeVersion":"v0",
	"chaincodeType": "golang",
	"args":["a","100","b","200"]
}'
```
成功返回结果
```
{"success":true,"message":"Successfully instantiate chaingcode in organization Org1 to the channel 'mychannel'"}
```
###### 7.4.4、数据上链
```
curl -s -X POST \
  http://localhost:4000/api/v1/save\
  -H "authorization: Bearer $ORG1_TOKEN" \
  -H "content-type: application/json" \
  -d '{
        "godsCode": "2018092900004",
        "jsonInfo":
          {
          "discern":"1",
          "godsIssuerName":"2",
          "godsIssuerCode":"3",
          "godsTypeName":"4",
          "godsMarketValue":"4",
          "companyName":"5",
          "issuePrice":"6",
          "godsCode":"7",
          "sort":"9",
          "orgCompanyName":"8",
          "orgCode":"",
          "reportId":"",
          "linkTime":"",
          "linkUser":"",
          "linkRole":"",
          "orgSignTime":"",
          "godsSignTime":"",
          "files":[{"fileName":"文件名称.xml","fileHash":"4564613243454546884464"}],
          "attr":[{"key":"name","value":"val"}],
          "companyArea":"所属国家/地区",
          "linkmanPhone":"企业联系电话",
          "linkmanEmail":"企业邮箱",
          "legalPerson":"法人姓名"
        } 
      }'
```
###### 7.4.5、数据查询
```
#通过区块查询
curl -s -X GET \
  "http://localhost:4000/channels/mychannel/blocks/1?peer=peer0.org1.example.com" \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json"

#通过TransactionID
curl -s -X GET \
http://localhost:4000/channels/mychannel/transactions/<put transaction id here>?peer=peer0.org1.example.com \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json"

#根据自己的业务ID查询
curl -s -X POST \
    "http://localhost:4000/api/v1/query" \
  -H "authorization: Bearer $ORG1_TOKEN" \
  -H "content-type: application/json" \
  -d '
    {
      "godsCode": "2018092900004"
    }
  '
```
返回结果
```
{"code":200,"message":"操作成功","data":[{"godsCode":"2018092900004","jsonInfo":{"companyArea":"所属国家/地区","companyName":"5","discern":"1","files":[{"fileHash":"4564613243454546884464","fileName":"文件名称.xml"}],"godsCode":"7","godsIssuerCode":"3","godsIssuerName":"2","godsMarketValue":"4","godsSignTime":"","godsTypeName":"4","issuePrice":"6","legalPerson":"法人姓名","linkRole":"","linkTime":"","linkUser":"","linkmanEmail":"企业邮箱","linkmanPhone":"企业联系电话","orgCode":"","orgCompanyName":"8","orgSignTime":"","reportId":"","sort":"9"},"userName":"doc"}]}
```
##### 未完待续。。。
#### 参考资料
1、https://github.com/hyperledger/fabric-samples/blob/release-1.3/balance-transfer/README.md  
2、https://jwt.io/  


