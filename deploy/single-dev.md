# 单机测试网络(byfn)

序号 | 内容 | 更新人 | 更新日期 | 版本
---| --- | --- | --- | ---
1 | 文档初始化 | 许向 | 2019-2-27 | 0.1

## 前提

在 [系统准备](../sysreqs.md#环境准备工作) 已经下载了二进制文件、镜像和samples

## 启动byfn

```
cd ~/fabric/fabric-samples/first-network
```

启动

```
./byfn.sh up -c demo1 -s couchdb
```

输入y确认

```
Starting for channel 'demo1' with CLI timeout of '10' seconds and CLI delay of '3' seconds and using database 'couchdb'
Continue? [Y/n]
```

参数用help来看含义，建议看一下byfn脚本的过程

[byfn 启动过程详解](byfn-startup-details.md)

## 测试

```
docker exec -ti cli peer chaincode query -C demo1 -n mycc -c '{"Args":["query","a"]}'
```

返回90，成功


## 参考
- [Writing Your First Application](https://hyperledger-fabric.readthedocs.io/en/latest/write_first_app.html)
