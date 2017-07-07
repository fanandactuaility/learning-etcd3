# Linux单机部署

> 注： 参考 [etcd releases页面](https://github.com/coreos/etcd/releases/) 的说明

## 下载

执行下面的命令，下载(大概10M)解压即可，无需安装：

```bash
curl -L https://github.com/coreos/etcd/releases/download/v3.2.1/etcd-v3.2.1-linux-amd64.tar.gz -o etcd-v3.2.1-linux-amd64.tar.gz
tar xzvf etcd-v3.2.1-linux-amd64.tar.gz
mv etcd-v3.2.1-linux-amd64 etcd
cd etcd
./etcd --version

etcd Version: 3.2.1
Git SHA: 61fc123
Go Version: go1.8.3
Go OS/Arch: linux/amd64

```

> 注： 以 etcd-v3.2.1 为例，后续更新版本时可能细节有所不同。

安装目录文件列表如下：

```bash
$ ls
default.etcd   etcd     README-etcdctl.md  READMEv2-etcdctl.md
Documentation  etcdctl  README.md
```

## 运行

直接运行命令 `./etcd` 就可以启动了，非常简单。

默认使用2379端口为客户端提供通讯， 并使用端口2380来进行服务器间通讯。

## 配置

为了方便使用，将 etcd 加入 PATH，另外设置 ETCDCTL_API 为3(后面解释)。

在 `/etc/profile` 中加入以下内容：

```bash
# etcd
export PATH=/home/sky/work/soft/etcd:$PATH
export ETCDCTL_API=3
```

然后执行 `source /etc/profile` 重新加载。

## 客户端访问

### 配置etcdctl

etcdctl 是 etcd 的客户端命令行。

> 特别提醒：使用 etcdctl 前，**务必设置环境变量 `ETCDCTL_API=3` **!

注意：如果不设置 `ETCDCTL_API=3`，则默认是的API版本是2：

```bash
$ etcdctl version
etcdctl version: 3.2.1
API version: 2
```

正确设置后，API版本变成3：

```bash
$ etcdctl version
etcdctl version: 3.2.1
API version: 3.2
```

### 使用etcdctl

通过下面的put和get命令来验证连接并操作etcd：

```bash
$ ./etcdctl put aaa 1
OK
$ ./etcdctl get aaa
aaa
1
```

## 总结

上面操作完成之后，就有一个可运行的简单 etcd 服务器和一个可用的 etcdctl 客户端。
