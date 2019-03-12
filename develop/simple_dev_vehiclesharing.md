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

... TODO

## 参考
- [Fabric 区块链智能合约 Chaincode 与客户端程序开发全新实例教程](https://www.ibm.com/developerworks/cn/cloud/library/cl-lo-hyperledger-fabric-practice-analysis2/index.html?ca=drs-)
- [Developing Applications](https://hyperledger-fabric.readthedocs.io/en/latest/developapps/developing_applications.html)
