# Introduction

序号 | 内容 | 更新人 | 更新日期 | 版本
---| --- | --- | --- | ---
1 | 文档初始化 | 许向 | 2019-2-26 | 0.1
2 | 添加内容 | 许向 | 2019-3-7 | 0.2

想在公司的某个项目的二期上使用Hyperledger Fabric作为底层

文档主要用于记录从部署到开发的一些坑，目录里面有些是直接跳的别人的文章。

为什么要用Fabric

项目目标考虑是toB的场景，根据Fabric的介绍

Hyperledger Fabric is an open source enterprise-grade permissioned distributed ledger technology (DLT) platform

Enterprise requirements:

- Participants must be identified/identifiable
- Networks need to be permissioned
- High transaction throughput performance
- Low latency of transaction confirmation
- Privacy and confidentiality of transactions and data pertaining to business transactions

## Fabric 1.4

1.4是Fabric第一个LTS的版本，LTS一般意味着这个版本可以被认可用于生产

2个新特性点

- 可服务性和操作性

   Fabric v1.4在日志记录改进、健康检查和操作指标方面取得了巨大的进步，提供了3个服务，用于监控和管理对等节点和订单节点

   - 日志/logspec端点

      允许操作人员动态获取和设置对等节点和订单节点的日志级别。

   - /healthz端点

      允许操作人员和容器编配服务检查对等节点和订单节点的活动性和健康状况。

   - metrics端点

      允许操作人员利用Prometheus从对等节点和订单节点提取操作指标。

- 改进了开发应用程序的应用开发模型
   这里的改进主要针对的是官方的fabric-nodejs-sdk，推荐使用官方的sdk开发，方便与现有的前端项目整合


## 参考
- [What’s new in v1.4](https://hyperledger-fabric.readthedocs.io/en/latest/whatsnew.html)
- [Fabric1.4来了...有哪些新特性？](https://zhuanlan.zhihu.com/p/52420063)
