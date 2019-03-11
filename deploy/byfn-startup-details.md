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

改写成我们自定义的crypto-config-bidpoc.yaml

```yaml
OrdererOrgs:
  - Name: Orderer
    Domain: union.bidpoc.com
    Specs:
      - Hostname: orderer
PeerOrgs:
  - Name: 9Chain
    Domain: 9chain.union.bidpoc.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 2
  - Name: Bidpoc
    Domain: bidpoc.union.bidpoc.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 2
  - Name: Artwook
    Domain: artwook.union.bidpoc.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 2
```


生成证书

```
cryptogen generate --config=./crypto-config-bidpoc.yaml
```

执行之后会在当前目录下创建一个crypto-config的目录，[完整目录树看这里](#crypto-config-directory)

探究一下内部的结构

crypto-config 分成2个子目录

```
crypto-config
├── ordererOrganizations
└── peerOrganizations
```

先看

```
crypto-config/ordererOrganizations/union.bidpoc.com/ca/
├── ca.union.bidpoc.com-cert.pem
└── daee5bdd4a3fa068ba0fca479f27bdca8db93631aabfe10362a4ce87f21ef876_sk
```

...TODO

## 参考
- [Hyperledger Fabric 实践与分析，第 1 部分](https://www.ibm.com/developerworks/cn/cloud/library/cl-lo-hyperledger-fabric-practice-analysis/index.html?ca=drs-)

## crypto-config-directory
```
crypto-config
├── ordererOrganizations
│   └── union.bidpoc.com
│       ├── ca
│       │   ├── ca.union.bidpoc.com-cert.pem
│       │   └── daee5bdd4a3fa068ba0fca479f27bdca8db93631aabfe10362a4ce87f21ef876_sk
│       ├── msp
│       │   ├── admincerts
│       │   │   └── Admin@union.bidpoc.com-cert.pem
│       │   ├── cacerts
│       │   │   └── ca.union.bidpoc.com-cert.pem
│       │   └── tlscacerts
│       │       └── tlsca.union.bidpoc.com-cert.pem
│       ├── orderers
│       │   └── orderer.union.bidpoc.com
│       │       ├── msp
│       │       │   ├── admincerts
│       │       │   │   └── Admin@union.bidpoc.com-cert.pem
│       │       │   ├── cacerts
│       │       │   │   └── ca.union.bidpoc.com-cert.pem
│       │       │   ├── keystore
│       │       │   │   └── 1a7178dc50bef02b3e8def47b4560bd99e6a1aac08d973379a97a74fa807e4d1_sk
│       │       │   ├── signcerts
│       │       │   │   └── orderer.union.bidpoc.com-cert.pem
│       │       │   └── tlscacerts
│       │       │       └── tlsca.union.bidpoc.com-cert.pem
│       │       └── tls
│       │           ├── ca.crt
│       │           ├── server.crt
│       │           └── server.key
│       ├── tlsca
│       │   ├── 1843c4f77280bf1d3664848a1ec83cab5eb694bb75c27e943b6201ac7e169f48_sk
│       │   └── tlsca.union.bidpoc.com-cert.pem
│       └── users
│           └── Admin@union.bidpoc.com
│               ├── msp
│               │   ├── admincerts
│               │   │   └── Admin@union.bidpoc.com-cert.pem
│               │   ├── cacerts
│               │   │   └── ca.union.bidpoc.com-cert.pem
│               │   ├── keystore
│               │   │   └── 57a239636f7fb3533a339647062ce4cc7000b6d12861097d72d43dfd381dde3e_sk
│               │   ├── signcerts
│               │   │   └── Admin@union.bidpoc.com-cert.pem
│               │   └── tlscacerts
│               │       └── tlsca.union.bidpoc.com-cert.pem
│               └── tls
│                   ├── ca.crt
│                   ├── client.crt
│                   └── client.key
└── peerOrganizations
    ├── 9chain.union.bidpoc.com
    │   ├── ca
    │   │   ├── 58810b70c53fd74636eb425b0a974c892d69dd45766e829f0c4e9df5626ee3ef_sk
    │   │   └── ca.9chain.union.bidpoc.com-cert.pem
    │   ├── msp
    │   │   ├── admincerts
    │   │   │   └── Admin@9chain.union.bidpoc.com-cert.pem
    │   │   ├── cacerts
    │   │   │   └── ca.9chain.union.bidpoc.com-cert.pem
    │   │   ├── config.yaml
    │   │   └── tlscacerts
    │   │       └── tlsca.9chain.union.bidpoc.com-cert.pem
    │   ├── peers
    │   │   ├── peer0.9chain.union.bidpoc.com
    │   │   │   ├── msp
    │   │   │   │   ├── admincerts
    │   │   │   │   │   └── Admin@9chain.union.bidpoc.com-cert.pem
    │   │   │   │   ├── cacerts
    │   │   │   │   │   └── ca.9chain.union.bidpoc.com-cert.pem
    │   │   │   │   ├── config.yaml
    │   │   │   │   ├── keystore
    │   │   │   │   │   └── 8d08701ee141afff997c74b9179b7407570c102cfee4528f37bb51d2b4f87c6d_sk
    │   │   │   │   ├── signcerts
    │   │   │   │   │   └── peer0.9chain.union.bidpoc.com-cert.pem
    │   │   │   │   └── tlscacerts
    │   │   │   │       └── tlsca.9chain.union.bidpoc.com-cert.pem
    │   │   │   └── tls
    │   │   │       ├── ca.crt
    │   │   │       ├── server.crt
    │   │   │       └── server.key
    │   │   └── peer1.9chain.union.bidpoc.com
    │   │       ├── msp
    │   │       │   ├── admincerts
    │   │       │   │   └── Admin@9chain.union.bidpoc.com-cert.pem
    │   │       │   ├── cacerts
    │   │       │   │   └── ca.9chain.union.bidpoc.com-cert.pem
    │   │       │   ├── config.yaml
    │   │       │   ├── keystore
    │   │       │   │   └── b8f1acc5da97d7c1348594f73a3f543baa29e69b407dd6f3860a21dc296c538a_sk
    │   │       │   ├── signcerts
    │   │       │   │   └── peer1.9chain.union.bidpoc.com-cert.pem
    │   │       │   └── tlscacerts
    │   │       │       └── tlsca.9chain.union.bidpoc.com-cert.pem
    │   │       └── tls
    │   │           ├── ca.crt
    │   │           ├── server.crt
    │   │           └── server.key
    │   ├── tlsca
    │   │   ├── f03dcb48ebea8ee9645d3616cf3e751e7b2e7dc901dfa8ba52fbc9268260f329_sk
    │   │   └── tlsca.9chain.union.bidpoc.com-cert.pem
    │   └── users
    │       ├── Admin@9chain.union.bidpoc.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   │   └── Admin@9chain.union.bidpoc.com-cert.pem
    │       │   │   ├── cacerts
    │       │   │   │   └── ca.9chain.union.bidpoc.com-cert.pem
    │       │   │   ├── keystore
    │       │   │   │   └── b805863731f4b9cede46f74bb2eccf614b097e683236645a388d2997ca0e711c_sk
    │       │   │   ├── signcerts
    │       │   │   │   └── Admin@9chain.union.bidpoc.com-cert.pem
    │       │   │   └── tlscacerts
    │       │   │       └── tlsca.9chain.union.bidpoc.com-cert.pem
    │       │   └── tls
    │       │       ├── ca.crt
    │       │       ├── client.crt
    │       │       └── client.key
    │       ├── User1@9chain.union.bidpoc.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   │   └── User1@9chain.union.bidpoc.com-cert.pem
    │       │   │   ├── cacerts
    │       │   │   │   └── ca.9chain.union.bidpoc.com-cert.pem
    │       │   │   ├── keystore
    │       │   │   │   └── 7eb8b39a096c81e57848e9a42c1b49a7cbe005e4d5c33ba1911af7b4eb0ceb70_sk
    │       │   │   ├── signcerts
    │       │   │   │   └── User1@9chain.union.bidpoc.com-cert.pem
    │       │   │   └── tlscacerts
    │       │   │       └── tlsca.9chain.union.bidpoc.com-cert.pem
    │       │   └── tls
    │       │       ├── ca.crt
    │       │       ├── client.crt
    │       │       └── client.key
    │       └── User2@9chain.union.bidpoc.com
    │           ├── msp
    │           │   ├── admincerts
    │           │   │   └── User2@9chain.union.bidpoc.com-cert.pem
    │           │   ├── cacerts
    │           │   │   └── ca.9chain.union.bidpoc.com-cert.pem
    │           │   ├── keystore
    │           │   │   └── 0613feaf11b8c7d976f35292ce5130a1ebc443d28739a9ffcc7c87de53f65da1_sk
    │           │   ├── signcerts
    │           │   │   └── User2@9chain.union.bidpoc.com-cert.pem
    │           │   └── tlscacerts
    │           │       └── tlsca.9chain.union.bidpoc.com-cert.pem
    │           └── tls
    │               ├── ca.crt
    │               ├── client.crt
    │               └── client.key
    ├── artwook.union.bidpoc.com
    │   ├── ca
    │   │   ├── 43c8c87224e39f079f85f0a3f44fc530d866d485bd88ab80953a75f484cc2171_sk
    │   │   └── ca.artwook.union.bidpoc.com-cert.pem
    │   ├── msp
    │   │   ├── admincerts
    │   │   │   └── Admin@artwook.union.bidpoc.com-cert.pem
    │   │   ├── cacerts
    │   │   │   └── ca.artwook.union.bidpoc.com-cert.pem
    │   │   ├── config.yaml
    │   │   └── tlscacerts
    │   │       └── tlsca.artwook.union.bidpoc.com-cert.pem
    │   ├── peers
    │   │   ├── peer0.artwook.union.bidpoc.com
    │   │   │   ├── msp
    │   │   │   │   ├── admincerts
    │   │   │   │   │   └── Admin@artwook.union.bidpoc.com-cert.pem
    │   │   │   │   ├── cacerts
    │   │   │   │   │   └── ca.artwook.union.bidpoc.com-cert.pem
    │   │   │   │   ├── config.yaml
    │   │   │   │   ├── keystore
    │   │   │   │   │   └── 190af020b2ba2963f91f182a71121fd3b621346a5981f8302337177455457079_sk
    │   │   │   │   ├── signcerts
    │   │   │   │   │   └── peer0.artwook.union.bidpoc.com-cert.pem
    │   │   │   │   └── tlscacerts
    │   │   │   │       └── tlsca.artwook.union.bidpoc.com-cert.pem
    │   │   │   └── tls
    │   │   │       ├── ca.crt
    │   │   │       ├── server.crt
    │   │   │       └── server.key
    │   │   └── peer1.artwook.union.bidpoc.com
    │   │       ├── msp
    │   │       │   ├── admincerts
    │   │       │   │   └── Admin@artwook.union.bidpoc.com-cert.pem
    │   │       │   ├── cacerts
    │   │       │   │   └── ca.artwook.union.bidpoc.com-cert.pem
    │   │       │   ├── config.yaml
    │   │       │   ├── keystore
    │   │       │   │   └── b34d0c5d53b90091a02b40824aac3e9f902e99a568fb95b9503b31dc1d7419b8_sk
    │   │       │   ├── signcerts
    │   │       │   │   └── peer1.artwook.union.bidpoc.com-cert.pem
    │   │       │   └── tlscacerts
    │   │       │       └── tlsca.artwook.union.bidpoc.com-cert.pem
    │   │       └── tls
    │   │           ├── ca.crt
    │   │           ├── server.crt
    │   │           └── server.key
    │   ├── tlsca
    │   │   ├── e205e3c3b7736b0cba515067fd69ed58ea08c20c124d008d5993412ff8cacf32_sk
    │   │   └── tlsca.artwook.union.bidpoc.com-cert.pem
    │   └── users
    │       ├── Admin@artwook.union.bidpoc.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   │   └── Admin@artwook.union.bidpoc.com-cert.pem
    │       │   │   ├── cacerts
    │       │   │   │   └── ca.artwook.union.bidpoc.com-cert.pem
    │       │   │   ├── keystore
    │       │   │   │   └── 4307f6862803cf4012e0781c298fc02f23140cc221001d82c6f382b2c02bf922_sk
    │       │   │   ├── signcerts
    │       │   │   │   └── Admin@artwook.union.bidpoc.com-cert.pem
    │       │   │   └── tlscacerts
    │       │   │       └── tlsca.artwook.union.bidpoc.com-cert.pem
    │       │   └── tls
    │       │       ├── ca.crt
    │       │       ├── client.crt
    │       │       └── client.key
    │       ├── User1@artwook.union.bidpoc.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   │   └── User1@artwook.union.bidpoc.com-cert.pem
    │       │   │   ├── cacerts
    │       │   │   │   └── ca.artwook.union.bidpoc.com-cert.pem
    │       │   │   ├── keystore
    │       │   │   │   └── ad73964b0b29a14ac2230354e79e00a864bed7c97a5a7fe0e99aebfb8ec4b10e_sk
    │       │   │   ├── signcerts
    │       │   │   │   └── User1@artwook.union.bidpoc.com-cert.pem
    │       │   │   └── tlscacerts
    │       │   │       └── tlsca.artwook.union.bidpoc.com-cert.pem
    │       │   └── tls
    │       │       ├── ca.crt
    │       │       ├── client.crt
    │       │       └── client.key
    │       └── User2@artwook.union.bidpoc.com
    │           ├── msp
    │           │   ├── admincerts
    │           │   │   └── User2@artwook.union.bidpoc.com-cert.pem
    │           │   ├── cacerts
    │           │   │   └── ca.artwook.union.bidpoc.com-cert.pem
    │           │   ├── keystore
    │           │   │   └── caaaf5c4a2d9f2eeba43f369b91de139176ccd799eb1cb94faf215fd2c32e92c_sk
    │           │   ├── signcerts
    │           │   │   └── User2@artwook.union.bidpoc.com-cert.pem
    │           │   └── tlscacerts
    │           │       └── tlsca.artwook.union.bidpoc.com-cert.pem
    │           └── tls
    │               ├── ca.crt
    │               ├── client.crt
    │               └── client.key
    └── bidpoc.union.bidpoc.com
        ├── ca
        │   ├── aefeea29f580bf0fcc85ba51705c7c6cf135c6cf8b32189ef289c55363ad3b00_sk
        │   └── ca.bidpoc.union.bidpoc.com-cert.pem
        ├── msp
        │   ├── admincerts
        │   │   └── Admin@bidpoc.union.bidpoc.com-cert.pem
        │   ├── cacerts
        │   │   └── ca.bidpoc.union.bidpoc.com-cert.pem
        │   ├── config.yaml
        │   └── tlscacerts
        │       └── tlsca.bidpoc.union.bidpoc.com-cert.pem
        ├── peers
        │   ├── peer0.bidpoc.union.bidpoc.com
        │   │   ├── msp
        │   │   │   ├── admincerts
        │   │   │   │   └── Admin@bidpoc.union.bidpoc.com-cert.pem
        │   │   │   ├── cacerts
        │   │   │   │   └── ca.bidpoc.union.bidpoc.com-cert.pem
        │   │   │   ├── config.yaml
        │   │   │   ├── keystore
        │   │   │   │   └── a213ba1e644bb91276e2b6b5ffa2079e038e24ddd8366451fda2606cb4349c36_sk
        │   │   │   ├── signcerts
        │   │   │   │   └── peer0.bidpoc.union.bidpoc.com-cert.pem
        │   │   │   └── tlscacerts
        │   │   │       └── tlsca.bidpoc.union.bidpoc.com-cert.pem
        │   │   └── tls
        │   │       ├── ca.crt
        │   │       ├── server.crt
        │   │       └── server.key
        │   └── peer1.bidpoc.union.bidpoc.com
        │       ├── msp
        │       │   ├── admincerts
        │       │   │   └── Admin@bidpoc.union.bidpoc.com-cert.pem
        │       │   ├── cacerts
        │       │   │   └── ca.bidpoc.union.bidpoc.com-cert.pem
        │       │   ├── config.yaml
        │       │   ├── keystore
        │       │   │   └── d780b85373e477fdf2f9cd8ec75de1f13e664208f6d1cbb0e3d51b5efd1fcc34_sk
        │       │   ├── signcerts
        │       │   │   └── peer1.bidpoc.union.bidpoc.com-cert.pem
        │       │   └── tlscacerts
        │       │       └── tlsca.bidpoc.union.bidpoc.com-cert.pem
        │       └── tls
        │           ├── ca.crt
        │           ├── server.crt
        │           └── server.key
        ├── tlsca
        │   ├── 94474aa340fb5091743b7e4300db7c82142abb8ee3ea85ff15a2ac1f8f343bac_sk
        │   └── tlsca.bidpoc.union.bidpoc.com-cert.pem
        └── users
            ├── Admin@bidpoc.union.bidpoc.com
            │   ├── msp
            │   │   ├── admincerts
            │   │   │   └── Admin@bidpoc.union.bidpoc.com-cert.pem
            │   │   ├── cacerts
            │   │   │   └── ca.bidpoc.union.bidpoc.com-cert.pem
            │   │   ├── keystore
            │   │   │   └── e8fe8a2fb46d50b3b93760d156007b1e16dfb1ba3d814bfdfe6481c078419ca4_sk
            │   │   ├── signcerts
            │   │   │   └── Admin@bidpoc.union.bidpoc.com-cert.pem
            │   │   └── tlscacerts
            │   │       └── tlsca.bidpoc.union.bidpoc.com-cert.pem
            │   └── tls
            │       ├── ca.crt
            │       ├── client.crt
            │       └── client.key
            ├── User1@bidpoc.union.bidpoc.com
            │   ├── msp
            │   │   ├── admincerts
            │   │   │   └── User1@bidpoc.union.bidpoc.com-cert.pem
            │   │   ├── cacerts
            │   │   │   └── ca.bidpoc.union.bidpoc.com-cert.pem
            │   │   ├── keystore
            │   │   │   └── df9d7138c0ed63d50dd7c05a1d278b543387aa6e82d3e88941908a2de5c3e6b9_sk
            │   │   ├── signcerts
            │   │   │   └── User1@bidpoc.union.bidpoc.com-cert.pem
            │   │   └── tlscacerts
            │   │       └── tlsca.bidpoc.union.bidpoc.com-cert.pem
            │   └── tls
            │       ├── ca.crt
            │       ├── client.crt
            │       └── client.key
            └── User2@bidpoc.union.bidpoc.com
                ├── msp
                │   ├── admincerts
                │   │   └── User2@bidpoc.union.bidpoc.com-cert.pem
                │   ├── cacerts
                │   │   └── ca.bidpoc.union.bidpoc.com-cert.pem
                │   ├── keystore
                │   │   └── de96320e77d299a39b5144e5119b8890e96fa872daf06f43ae993f4428934dab_sk
                │   ├── signcerts
                │   │   └── User2@bidpoc.union.bidpoc.com-cert.pem
                │   └── tlscacerts
                │       └── tlsca.bidpoc.union.bidpoc.com-cert.pem
                └── tls
                    ├── ca.crt
                    ├── client.crt
                    └── client.key

174 directories, 173 files
```
