安装
=======

## linux安装

> 注： 参考 [etcd releases页面](https://github.com/coreos/etcd/releases/) 的说明

执行下面的命令，下载(大概10M)解压：

```bash
curl -L https://github.com/coreos/etcd/releases/download/v3.0.2/etcd-v3.0.2-linux-amd64.tar.gz -o etcd-v3.0.2-linux-amd64.tar.gz

tar xzvf etcd-v3.0.2-linux-amd64.tar.gz

cd etcd-v3.0.2-linux-amd64
```

安装目录文件列表如下：

```bash
$ ls
default.etcd   etcd     README-etcdctl.md  READMEv2-etcdctl.md
Documentation  etcdctl  README.md
```

## 运行

直接运行命令 `./etcd` 就可以启动了，非常简单：

    $ ./etcd
    2016-07-14 18:23:10.185285 I | etcdmain: etcd Version: 3.0.2
    2016-07-14 18:23:10.185337 I | etcdmain: Git SHA: faeeb2f
    2016-07-14 18:23:10.185352 I | etcdmain: Go Version: go1.6.2
    2016-07-14 18:23:10.185366 I | etcdmain: Go OS/Arch: linux/amd64
    2016-07-14 18:23:10.185381 I | etcdmain: setting maximum number of CPUs to 8, total number of available CPUs is 8
    2016-07-14 18:23:10.185399 W | etcdmain: no data-dir provided, using default data-dir ./default.etcd
    2016-07-14 18:23:10.185666 I | etcdmain: listening for peers on http://localhost:2380
    2016-07-14 18:23:10.185724 I | etcdmain: listening for client requests on localhost:2379
    2016-07-14 18:23:10.188681 I | etcdserver: name = default
    2016-07-14 18:23:10.188706 I | etcdserver: data dir = default.etcd
    2016-07-14 18:23:10.188717 I | etcdserver: member dir = default.etcd/member
    2016-07-14 18:23:10.188726 I | etcdserver: heartbeat = 100ms
    2016-07-14 18:23:10.188735 I | etcdserver: election = 1000ms
    2016-07-14 18:23:10.188744 I | etcdserver: snapshot count = 10000
    2016-07-14 18:23:10.188757 I | etcdserver: advertise client URLs = http://localhost:2379
    2016-07-14 18:23:10.188783 I | etcdserver: initial advertise peer URLs = http://localhost:2380
    2016-07-14 18:23:10.188807 I | etcdserver: initial cluster = default=http://localhost:2380
    2016-07-14 18:23:10.192059 I | etcdserver: starting member 8e9e05c52164694d in cluster cdf818194e3a8c32
    2016-07-14 18:23:10.192117 I | raft: 8e9e05c52164694d became follower at term 0
    2016-07-14 18:23:10.192137 I | raft: newRaft 8e9e05c52164694d [peers: [], term: 0, commit: 0, applied: 0, lastindex: 0, lastterm: 0]
    2016-07-14 18:23:10.192150 I | raft: 8e9e05c52164694d became follower at term 1
    2016-07-14 18:23:10.205351 I | etcdserver: starting server... [version: 3.0.2, cluster version: to_be_decided]
    2016-07-14 18:23:10.207473 I | membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32
    2016-07-14 18:23:10.392465 I | raft: 8e9e05c52164694d is starting a new election at term 1
    2016-07-14 18:23:10.392515 I | raft: 8e9e05c52164694d became candidate at term 2
    2016-07-14 18:23:10.392526 I | raft: 8e9e05c52164694d received vote from 8e9e05c52164694d at term 2
    2016-07-14 18:23:10.392547 I | raft: 8e9e05c52164694d became leader at term 2
    2016-07-14 18:23:10.392561 I | raft: raft.node: 8e9e05c52164694d elected leader 8e9e05c52164694d at term 2
    2016-07-14 18:23:10.392834 I | etcdserver: setting up the initial cluster version to 3.0
    2016-07-14 18:23:10.394040 N | membership: set the initial cluster version to 3.0
    2016-07-14 18:23:10.394128 I | etcdserver: published {Name:default ClientURLs:[http://localhost:2379]} to cluster cdf818194e3a8c32
    2016-07-14 18:23:10.394143 I | etcdmain: ready to serve client requests
    2016-07-14 18:23:10.394455 N | etcdmain: serving insecure client requests on localhost:2379, this is strongly discouraged!
    2016-07-14 18:23:10.707654 I | api: enabled capabilities for version 3.0

默认使用2379端口为客户端提供通讯， 并使用端口2380来进行服务器间通讯。

查看当前安装的版本：

    $ ./etcd --version
    etcd Version: 3.0.2
    Git SHA: faeeb2f
    Go Version: go1.6.2
    Go OS/Arch: linux/amd64

## 客户端访问

### 配置etcdctl

etcdctl 是 etcd 的客户端命令行。

特别提醒：使用前，**务必设置环境变量 `ETCDCTL_API=3` **!

在 `/etc/profile` 中加入以下内容：

```bash
# etcd
export PATH=/home/sky/work/soft/etcd:$PATH
export ETCDCTL_API=3
```

然后执行 `source /etc/profile` 重新加载。

注意：如果不设置 `ETCDCTL_API=3`，则默认是的API版本是2：

```bash
$ ./etcdctl --version
etcdctl version: 3.0.2
API version: 2
```

正确设置后，API版本变成3：

```bash
$ etcdctl version
etcdctl version: 3.0.2
API version: 3.0
```

### 使用etcdctl

通过下面的put和get命令来验证连接并操作etcd：

```bash
$ ./etcdctl --endpoints=[127.0.0.1:2379] put aaa 1
OK
$ ./etcdctl --endpoints=127.0.0.1:2379 get aaa
aaa
1
```

这里的 `--endpoints=[127.0.0.1:2379]` 用于指定连接的目标etcd地址。

有个非常令人疑惑的地方，如果不指定endpoints，默认居然是连接 `127.0.0.1:22379`, 然后报错说连不上,因为etcd默认启动时使用的端口是2379：

```bash
$./etcdctl get aaa

2016/07/15 11:10:40 grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp 127.0.0.1:22379: getsockopt: connection refused"; Reconnecting to {"127.0.0.1:22379" <nil>}
Error:  grpc: timed out when dialing
```

## 修改etcd启动配置

除了上述指定endpoints的方法(每次都要指定，太繁琐，不是个好办法)之外，还有一个解决方案就是启动etcd时增加endpoints：

```bash
./etcd -listen-client-urls http://127.0.0.1:2379,http://127.0.0.1:22379,http://127.0.0.1:32379 --advertise-client-urls=http://127.0.0.1:2380
```

注意上面的参数列表：

1. -listen-client-urls 用于指定etcd和客户端的连接端口，这里 2239/22239/32239 三个端口都要指定，只要缺一个 etcdcrl 就会报错
2. --advertise-client-urls 用于指定etcd服务器之间通讯的端口，etcd有要求，如果-listen-client-urls被设置了，那么就必须同时设置--advertise-client-urls，所以即使设置和默认相同，也必须显式设置。

## 总结

上面操作完成之后，就有一个可运行的简单 etcd 服务器和一个可用的 etcdctl。


