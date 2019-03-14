# 客户端开发实例 - Vehicle Sharing

序号 | 内容 | 更新人 | 更新日期 | 版本
---| --- | --- | --- | ---
1 | 文档初始化 | 许向 | 2019-3-4 | 0.1

## 项目介绍

项目原型参考了[IBM-徐春雷-Hyperledger Fabric 实践与分析](https://www.ibm.com/developerworks/cn/cloud/library/cl-lo-hyperledger-fabric-practice-analysis2/index.html?ca=drs-)的系列文章，下面基础部分围绕此基准展开

## 环境准备

在 [系统准备](../sysreqs.md#环境准备工作) 已经下载了二进制文件、镜像和samples

## 获取代码

```bash
cd $HOME/fabric && git clone https://github.com/tomxucnxa/vehiclesharing.git
```

切换分支

```bash
cd $HOME/fabric/vehiclesharing && git checkout pub_1.4_2.2
```

确认是否成功

```bash
[ $(git status | grep -c '^On branch pub_1.4_2.2') -eq 1 ] && echo "OK" || echo "Failed"
```

复制代码

```bash
cd $HOME/fabric
cp -R vehiclesharing/chaincode/ fabric-samples/chaincode/vehiclesharing/
cp vehiclesharing/utils/setparas.sh fabric-samples/first-network/scripts/
chmod 755 fabric-samples/first-network/scripts/setparas.sh
```

## 启动first-network

***确认系统是干净的***

```bash
cd $HOME/fabric/fabric-samples/first-network && \
./byfn.sh generate && \
./byfn.sh up -c mychannel -s couchdb
```

## 安装Chaincode

### 登陆cli容器

```bash
docker exec -it cli bash


```
### 设置环境变量

```bash
export CC_SRC_PATH="github.com/chaincode/vehiclesharing"
export VERSION="1.0"
export CC_NAME="vehiclesharing"
source ./scripts/setparas.sh
```

这里的VERSION 1.0指的是vehiclesharing这个项目的版本

看下./scripts/setparas.sh

```bash
CRYPTO_PATH="/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto"

export CHANNEL_NAME="mychannel"
export LANGUAGE="golang"
export ORDERER_CA=${CRYPTO_PATH}/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export ORDERER_ADDRESS="orderer.example.com:7050"
export DEFAULT_POLICY="AND ('Org1MSP.peer','Org2MSP.peer')"

# To export the environment variables.
exportPeerEnv() {
    peer=$1
    org=$2
    export CORE_PEER_TLS_ROOTCERT_FILE="${CRYPTO_PATH}/peerOrganizations/org${org}.example.com/peers/peer${peer}.org${org}.example.com/tls/ca.crt"
    export CORE_PEER_MSPCONFIGPATH="${CRYPTO_PATH}/peerOrganizations/org${org}.example.com/users/Admin@org${org}.example.com/msp"
    export CORE_PEER_LOCALMSPID="Org${org}MSP"
    export CORE_PEER_ADDRESS="peer${peer}.org${org}.example.com:7051"
    export CORE_PEER_TLS_CERT_FILE="${CRYPTO_PATH}/peerOrganizations/org${org}.example.com/peers/peer${peer}.org${org}.example.com/tls/server.crt"
    export CORE_PEER_TLS_KEY_FILE="${CRYPTO_PATH}/peerOrganizations/org${org}.example.com/peers/peer${peer}.org${org}.example.com/tls/server.key"

    echo "These CORE_PEER environment variables are set:"
    env | grep CORE_PEER
}

# To set the peer connection parameters. It is only for the fixed 2 peers, 2 orgs in the example.
exportPeerConn2Nodes() {
    peer=$1
    org=$2
    peerSecond=$3
    orgSecond=$4
    export PEER_CONN_PARMS="--peerAddresses peer${peer}.org${org}.example.com:7051 --tlsRootCertFiles ${CRYPTO_PATH}/peerOrganizations/org${org}.example.com/peers/peer${peer}.org${org}.example.com/tls/ca.crt --peerAddresses peer${peerSecond}.org${orgSecond}.example.com:7051 --tlsRootCertFiles ${CRYPTO_PATH}/peerOrganizations/org${orgSecond}.example.com/peers/peer${peerSecond}.org${orgSecond}.example.com/tls/ca.crt"

    echo "The PEER_CONN_PARMS is set:"
    echo $PEER_CONN_PARMS
}

opt=$1
peer=$2
org=$3
peerSecond=$4
orgSecond=$5

if [ "${opt}" == "peerenv" ]; then
  exportPeerEnv ${peer} ${org}
elif [ "${opt}" == "peerconn" ]; then
  exportPeerConn2Nodes ${peer} ${org} ${peerSecond} ${orgSecond}
else
  echo "The common environment variables are exported."
fi
```

脚本接收5个参数
- $1 是子命令 [ peerenv | peerconn ]
- $2 是peer的序号(根据byfn的脚本)
- $3 是org的序号
- $4 第二个peer的序号
- $5 第二个org的序号

example:

```bash
source ./scripts/setparas.sh peerenv 0 1
```

相当于执行了

```bash
export CORE_PEER_TLS_ROOTCERT_FILE="/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
export CORE_PEER_MSPCONFIGPATH="/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp"
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_ADDRESS="peer0.org1.example.com:7051"
export CORE_PEER_TLS_CERT_FILE="/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt"
export CORE_PEER_TLS_KEY_FILE="/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key"
```

### 安装chaincode

```bash
source ./scripts/setparas.sh peerenv 0 1
peer chaincode install -n $CC_NAME -v $VERSION -l $LANGUAGE -p $CC_SRC_PATH

source ./scripts/setparas.sh peerenv 0 2
peer chaincode install -n $CC_NAME -v $VERSION -l $LANGUAGE -p $CC_SRC_PATH

source ./scripts/setparas.sh peerenv 1 1
peer chaincode install -n $CC_NAME -v $VERSION -l $LANGUAGE -p $CC_SRC_PATH

source ./scripts/setparas.sh peerenv 1 2
peer chaincode install -n $CC_NAME -v $VERSION -l $LANGUAGE -p $CC_SRC_PATH

peer chaincode instantiate -o $ORDERER_ADDRESS --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n $CC_NAME -l $LANGUAGE -v $VERSION -c '{"Args":[""]}' -P "$DEFAULT_POLICY"
```

- install 安装代码
- instantiate 实例化代码


### 更新chaincode

更新和安装类似，VERSION 要不同于之前的版本


```bash
export CC_SRC_PATH="github.com/chaincode/vehiclesharing"
export VERSION="1.1"
export CC_NAME="vehiclesharing"
source ./scripts/setparas.sh

source ./scripts/setparas.sh peerenv 0 1
peer chaincode install -n $CC_NAME -v $VERSION -l $LANGUAGE -p $CC_SRC_PATH

source ./scripts/setparas.sh peerenv 0 2
peer chaincode install -n $CC_NAME -v $VERSION -l $LANGUAGE -p $CC_SRC_PATH

source ./scripts/setparas.sh peerenv 1 1
peer chaincode install -n $CC_NAME -v $VERSION -l $LANGUAGE -p $CC_SRC_PATH

source ./scripts/setparas.sh peerenv 1 2
peer chaincode install -n $CC_NAME -v $VERSION -l $LANGUAGE -p $CC_SRC_PATH

peer chaincode upgrade -o $ORDERER_ADDRESS --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n $CC_NAME -v $VERSION -c '{"Args":[""]}' -P "$DEFAULT_POLICY"

```


## 调用chaincode

### cli节点中调用

进入cli节点

```bash
docker exec -it cli bash
export CC_SRC_PATH="github.com/chaincode/vehiclesharing"
export VERSION="1.1"
export CC_NAME="vehiclesharing"
source ./scripts/setparas.sh
```

- 添加 Vehicle

   ```bash
   source ./scripts/setparas.sh peerconn 0 1 0 2
   peer chaincode invoke -o $ORDERER_ADDRESS --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n $CC_NAME $PEER_CONN_PARMS -c '{"Args":["createVehicle", "C123", "FBC"]}'
   ```

   连接参数如下
   ```
   --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
   ```

   调用的是org1.peer0与org2.peer0

- 查询 Vehicle

   ```bash
   source ./scripts/setparas.sh peerenv 0 1
   peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["findVehicle","C123"]}'
   ```

   系统返回

   ```
   {"brand":"FBC","createTime":0,"id":"C123","objectType":"VEHICLE","ownerId":"","price":0,"status":0,"userId":""}
   ```

   扩充

   查询org1-peer1行不行
   ```bash
   source ./scripts/setparas.sh peerenv 0 2
   peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["findVehicle","C123"]}'
   ```

   测试发现是可以的

   那么org2-peer0呢
   ```bash
   source ./scripts/setparas.sh peerenv 1 1
   peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["findVehicle","C123"]}'
   ```

   笔记本在卡了一会儿之后发现也有了输出

   这时再检查了一下docker容器

   ```bash
   docker ps --format "{{.Names}}: {{.Command}}" | grep 'vehiclesharing'
   ```

   发现这个时候有4个chaincode容器在运行

   ```
   dev-peer1.org1.example.com-vehiclesharing-1.0: "chaincode -peer.add…"
   dev-peer0.org2.example.com-vehiclesharing-1.0: "chaincode -peer.add…"
   dev-peer0.org1.example.com-vehiclesharing-1.0: "chaincode -peer.add…"
   dev-peer1.org2.example.com-vehiclesharing-1.0: "chaincode -peer.add…"
   ```

   看一下具体的调用，chaincode默认调用的是Invoke方法

   稍微看一下这个chaincode的函数签名

   ```golang
   func (t *VehicleSharing) Invoke(stub shim.ChaincodeStubInterface) peer.Response
   ```

   Invoke接收了一个实现了shim.ChaincodeStubInterface接口的结构，具体这个接口的实现函数列表可以参考
   `$GOPATH/src/github.com/hyperledger/fabric/core/chaincode/shim/interfaces.go`

   先往下看

   ```golang
    fn, args := stub.GetFunctionAndParameters()
	var res string
	var err error

	fn = strings.TrimSpace(fn)
	log.Printf("Invoke %s %v", fn, args)

	var FUNCMAP = map[string]func(shim.ChaincodeStubInterface, []string) (string, error){
		// Vehicle functions
		"createVehicle":         createVehicle,
		"findVehicle":           findVehicle,
		"deleteVehicle":         deleteVehicle,
		"updateVehiclePrice":    updateVehiclePrice,
		"updateVehicleDynPrice": updateVehicleDynPrice,
		"queryVehiclesByBrand":  queryVehiclesByBrand,
		"queryVehicles":         queryVehicles,
		"getVehicleHistory":     getVehicleHistory,
		"wronglyTxRandValue":    wronglyTxRandValue,
		// Lease functions
		"createLease": createLease,
		"findLease":   findLease}

	var ccAction = FUNCMAP[fn]
	if ccAction == nil {
		var errStr = fmt.Sprintf("Function %s doesn't exist.", fn)
		log.Printf(errStr)
		return shim.Error(errStr)
	}

	// TODO To handle the function if it doesn't exist.
	res, err = ccAction(stub, args)
   ```

   再看下传入的参数
   ```
   '{"Args":["findVehicle","C123"]}'
   ```

   ```golang
   fn, args := stub.GetFunctionAndParameters()
   ```

   通过这个方法，我们获取了2个参数，一个是fn，这里就是findVehicle(`string`)，另外一个就是['C123']\(`[]string`)

   然后代码中事先创建了一个字典，key是string，value是一个匿名函数，接收2个参数，一个是满足shim.ChaincodeStubInterface接口类型的结构，一个是[]string

   接下来根据传递进来的fn名来定位到某个方法，然后调用

   这里实际调用的是findVehicle(['C123'])


   根据 Brand 查询 Vehicle
   (任意节点上)
   ```
   source ./scripts/setparas.sh peerenv 1 2
   peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["queryVehiclesByBrand","FBC"]}'
   ```

   返回

   ```
   [{"brand":"FBC","createTime":0,"id":"C123","objectType":"VEHICLE","ownerId":"","price":123.55,"status":0,"userId":""}]
   ```

   看一下这个函数 queryVehiclesByBrand
   ```golang
   func queryVehiclesByBrand(stub shim.ChaincodeStubInterface, args []string) (string, error) {
       if len(args) < 1 {
           return "", fmt.Errorf("There should be at least one arg of queryVehiclesByBrand.")
       }
       // There should be only 1 arg for brand.
       var brand = strings.TrimSpace(args[0])
       if brand == "" {
           return "", fmt.Errorf("The brand cannot be blank.")
       }
       return queryVehicles(stub, []string{fmt.Sprintf(`{"selector":{"%s":"%s","brand":"%s"}}`, OBJECTTYPE_JSON, OBJECTTYPE_VEHICLE, brand)})
   }
   ```

   直接看最后，实际调用了queryVehicles。

   queryVehicles调用

   abric 内嵌的默认 State Database 是 LevelDB，它提供了简单的 Key-Value pairs 操作。CouchDB 则对 JSON 格式数据提供了很好支持，并支持 Rich Query。

   在启动 byfn 网络时，我们使用了 -s couchdb 参数，根据 docker yaml 文件中的定义，每个 Peer 都会连接一个独立的 CouchDB 数据库。

   queryVehicles这个函数实际是查询了CouchDB

   查询条件：objectType="VEHICLE"

   ```bash
   source ./scripts/setparas.sh peerenv 1 2
   peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["queryVehicles","{\"selector\":{\"objectType\":\"VEHICLE\"}}"]}'
   ```

   返回一个json数组
   ```json
   [{"brand":"FBC","createTime":0,"id":"C123","objectType":"VEHICLE","ownerId":"","price":123.55,"status":0,"userId":""}]
   ```

   查询条件：objectType="VEHICLE" and price < 200

   ```bash
   source ./scripts/setparas.sh peerenv 1 2
   peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["queryVehicles","{\"selector\":{\"objectType\":\"VEHICLE\",\"price\":{\"$lt\":200}}}"]}'

   ```

   返回一个json数组
   ```json
   [{"brand":"FBC","createTime":0,"id":"C123","objectType":"VEHICLE","ownerId":"","price":123.55,"status":0,"userId":""}]
   ```


 - 更新 Vehicle 价格

   前面我们在org1-peer0和org1-peer1上添加了C123，发现在所有节点上都可以查询到。

   原本教程是继续对org1-peer0和org1-peer1更新C123，这里我改一下，对org2-peer0和org2-peer1进行更新，看看会发生什么

   ```bash
   source ./scripts/setparas.sh peerconn 1 1 1 2
   peer chaincode invoke -o $ORDERER_ADDRESS --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n $CC_NAME $PEER_CONN_PARMS -c '{"Args":["updateVehiclePrice", "C123", "123.55"]}'
   ```

   系统返回

   ```
   2019-03-14 08:33:19.313 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200 payload:"C123"
   ```

   成功了，然后分别去4个peer上查询一下

   ```
   source ./scripts/setparas.sh peerenv 0 1
   peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["findVehicle","C123"]}'
   ```

   ```
   source ./scripts/setparas.sh peerenv 0 2
   peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["findVehicle","C123"]}'
   ```

   ```
   source ./scripts/setparas.sh peerenv 1 1
   peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["findVehicle","C123"]}'
   ```

   ```
   source ./scripts/setparas.sh peerenv 1 2
   peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["findVehicle","C123"]}'
   ```

   均返回了同样的

   ```
   {"brand":"FBC","createTime":0,"id":"C123","objectType":"VEHICLE","ownerId":"","price":123.55,"status":0,"userId":""}
   ```







... TODO

## 参考
- [Fabric 区块链智能合约 Chaincode 与客户端程序开发全新实例教程](https://www.ibm.com/developerworks/cn/cloud/library/cl-lo-hyperledger-fabric-practice-analysis2/index.html?ca=drs-)
- [Developing Applications](https://hyperledger-fabric.readthedocs.io/en/latest/developapps/developing_applications.html)
