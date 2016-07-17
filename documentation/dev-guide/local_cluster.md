# 搭建本地集群

> 注： 内容翻译自 [Setting up local clusters](https://github.com/coreos/etcd/blob/master/Documentation/dev-guide/local_cluster.md)

对于测试和开发部署，最快最简单的方式是搭建本地集群。对于产品部署，参考 [集群](clustering.md) 章节。

## 本地单独集群

部署etcd集群作为单独集群是直截了当的。仅用一个命令启动它：

```
$ ./etcd
...
```

启动的etcd成员在 `localhost:2379` 监听客户端请求。

通过使用 etcdctl 来和已经启动的集群交互：

```
# use API version 3
$ export ETCDCTL_API=3

$ ./etcdctl put foo bar
OK

$ ./etcdctl get foo
bar
```

> 注：如果遇到连接不上的问题，参考 [安装](../../installation/index.md) 一节中的说明。

## 本地多成员集群

提供Procfile 用于简化搭建本地多成员集群。通过少量命令来启动多成员集群：

```
# install goreman program to control Profile-based applications.
$ go get github.com/mattn/goreman
$ goreman -f Procfile start
...
```

> 注： 必须先安装 go，请见章节 [Go语言安装](../../installation/golang.md)

启动的成员各自在 `localhost:12379`, `localhost:22379`, 和 `localhost:32379` 上监听客户端请求。

通过使用 etcdctl 来和已经启动的集群交互：

```
# use API version 3
$ export ETCDCTL_API=3

$ etcdctl --write-out=table --endpoints=localhost:12379 member list
+------------------+---------+--------+------------------------+------------------------+
|        ID        | STATUS  |  NAME  |       PEER ADDRS       |      CLIENT ADDRS      |
+------------------+---------+--------+------------------------+------------------------+
| 8211f1d0f64f3269 | started | infra1 | http://127.0.0.1:12380 | http://127.0.0.1:12379 |
| 91bc3c398fb3c146 | started | infra2 | http://127.0.0.1:22380 | http://127.0.0.1:22379 |
| fd422379fda50e48 | started | infra3 | http://127.0.0.1:32380 | http://127.0.0.1:32379 |
+------------------+---------+--------+------------------------+------------------------+

$ etcdctl --endpoints=localhost:12379 put foo bar
OK
```

为了体验etcd的容错性，杀掉一个成员：
```
# kill etcd2
$ goreman run stop etcd2

$ etcdctl --endpoints=localhost:12379 put key hello
OK

$ etcdctl --endpoints=localhost:12379 get key
hello

# try to get key from the killed member
$ etcdctl --endpoints=localhost:22379 get key
2016/04/18 23:07:35 grpc: Conn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 127.0.0.1:22379: getsockopt: connection refused"; Reconnecting to "localhost:22379"
Error:  grpc: timed out trying to connect

# restart the killed member
$ goreman run restart etcd2

# get the key from restarted member
$ etcdctl --endpoints=localhost:22379 get key
hello
```

要学习更多和etcd的交互，请阅读 [和etcd交互](interacting_v3.md)


