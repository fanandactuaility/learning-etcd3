官方文档
======

> 注： 内容翻译自 https://github.com/coreos/etcd/blob/master/Documentation/docs.md

etcd 是一个分布式键值对存储，设计用来可靠而快速的保存关键数据并提供访问。通过分布式锁，leader选举和写屏障(write barriers)来开启可靠的分布式协同。etcd集群是为高可用，持久性数据存储和检索而准备。

## 开始

现在etcd的用户和开发者可以从 [下载并构建](https://github.com/coreos/etcd/blob/master/Documentation/dl_build.md) etcd开始。在获取etcd之后，跟随 [quick demo](https://github.com/coreos/etcd/blob/master/Documentation/demo.md) 来看构建和操作etcd集群的基本内容。

## 使用etcd开发

开始使用etcd作为分布式键值对存储的最简单的方式是 [搭建本地集群](dev-guide/local_cluster.md)

- [搭建本地集群](dev-guide/local_cluster.md)
- [和etcd交互](dev-guide/interacting_v3.md)