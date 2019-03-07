# byfn 启动过程详解

序号 | 内容 | 更新人 | 更新日期 | 版本
---| --- | --- | --- | ---
1 | 文档初始化 | 许向 | 2019-3-7 | 0.1

## 目录树
项目位于`$HOME/fabric`下

```bash
cd fabric/fabric-samples/first-network
```

## 清理环境

```bash
./byfn.sh down
```

## 设置环境变量

```bash
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}
export CLI_TIMEOUT=10
export CLI_DELAY=3
export CHANNEL_NAME="mychannel"
export COMPOSE_FILE=docker-compose-cli.yaml
export COMPOSE_FILE_COUCH=docker-compose-couch.yaml
export COMPOSE_FILE_ORG3=docker-compose-org3.yaml
export LANGUAGE=golang
export IMAGETAG="latest"
export VERBOSE=true
```

CHANNEL_NAME 是所要创建的频道的名字

## 证书

证书（certificate）是Fabric中权限管理的基础。目前采用了基于ECDSA算法的非对称加密算法来生成公钥和私钥，证书格式则采用了X.509的标准规范。

Fabric中采用单独的Fabric CA项目来管理证书的生成。每一个实体、组织都可以拥有自己的身份证书，并且证书也遵循了组织结构，方便基于组织实现灵活的权限管理。

生成的证书列表

- 自签名 CA 根证书
- 节点 (Orderer、Peer) 证书
- 管理员证书与证书对应的私钥
- TLS 证书

使用cryptogen命令生成证书的时候需要配置一个配置参数文件`crypto-config.yaml`

探索一下这个配置文件

```bash
grep -v -e '^\s*#' -v -e '^\s*$' crypto-config.yaml
```

内容如下

```yaml
OrdererOrgs:
  - Name: Orderer
    Domain: example.com
    Specs:
      - Hostname: orderer
PeerOrgs:
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
  - Name: Org2
    Domain: org2.example.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
```


...TODO

## 参考
- [Hyperledger Fabric 实践与分析，第 1 部分](https://www.ibm.com/developerworks/cn/cloud/library/cl-lo-hyperledger-fabric-practice-analysis/index.html?ca=drs-)
