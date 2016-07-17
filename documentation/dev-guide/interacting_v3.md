# 和etcd交互

> 注： 内容翻译自 [Interacting with etcd](https://github.com/coreos/etcd/blob/master/Documentation/dev-guide/interacting_v3.md)

用户常用通过放置或者获取key的值来和etcd交互。这一节描述如何使用etcdctl来操作， etcdctl是一个和etcd服务器交互的命令行工具。这里描述的概念也适用于gRPC API或者客户端类库API。

默认，为了向后兼容etcdctl使用v2 API来和etcd server通讯。为了让etcdctl使用v3 API来和etcd通讯，API版本必须通过环境变量 `ETCDCTL_API` 设置为 版本3。

``` bash
export ETCDCTL_API=3
```

## 写入key

应用通过写入key来储存key到etcd中。每个存储的key被通过Raft协议复制到所有etcd集群成员来达到一致性和可靠性。

这是设置key `foo` 的值为 `bar` 的命令:


``` bash
$ etcdctl put foo bar
OK
```

## 读取key

应用可以从etcd集群中读取key的值。查询可以读取单个key，或者某个范围的key。

假设etcd集群存储有下面的key：

```
foo = bar
foo1 = bar1
foo3 = bar3
```

这是读取key `for` 的值的命令：

```bash
$ etcdctl get foo
foo
bar
```

这是覆盖从 `foo` to `foo9` 的key的命令：

```bash
$ etcdctl get foo foo9
foo
bar
foo1
bar1
foo3
bar3
```

> 注： 测试中发现，这个命令的有效区间是这样：`[foo foo9)`， 即包含第一个参数，但是不包含第二个参数。因此如果第二个参数是 `foo3`，上面的命令是不会返回 key `foo3` 的值的。

## 读取key过去版本的值

应用可能想读取key的被替代的值。例如，应用可能想通过访问key的先前版本来回滚到旧的配置。或者，应用可能想通过访问key历史记录的多个请求来得到一个覆盖多个key上的统一视图(TBD: 没看懂:))。

因为etcd集群上键值存储的每个修改都会增加etcd集群的全局修订版本(revision)，应用可以通过提供旧有的etcd版本来读取被替代的key。

假设etcd集群已经有下列key：

``` bash
$ etcdctl put foo bar         # revision = 2
$ etcdctl put foo1 bar1       # revision = 3
$ etcdctl put foo bar_new     # revision = 4
$ etcdctl put foo1 bar1_new   # revision = 5
```

这里是访问key的过去版本的例子：

```bash
$ etcdctl get foo foo9 # access the most recent versions of keys
foo
bar_new
foo1
bar1_new

$ etcdctl get --rev=4 foo foo9 # access the versions of keys at revision 4
foo
bar_new
foo1
bar1

$ etcdctl get --rev=3 foo foo9 # access the versions of keys at revision 3
foo
bar
foo1
bar1

$ etcdctl get --rev=2 foo foo9 # access the versions of keys at revision 2
foo
bar

$ etcdctl get --rev=1 foo foo9 # access the versions of keys at revision 1
```

> 注： 有个疑问，怎么知道当前etcd的revision是多少？新安装的etcd我是靠一个一个值推断出来的，但是这肯定不是个好办法。TBD....

## 删除key

应用可以从etcd集群中删除一个key或者特定范围的key。

下面是删除key `foo` 的命令：

```bash
$ etcdctl del foo
1 # 删除了一个key
```
这是删除从 `foo` to `foo9` 范围的key的命令：

```bash
$ etcdctl del foo foo9
2 # 删除了两个key
```

> 注： 这里有个奇怪的事情，如果用命令 `etcdctl del foo foo10`,是可以删除key foo/foo1/foo2 的，但是无法删除 foo3，换成命令 `etcdctl del foo foo9` 又可以删除 foo3。这个规律....

## 观察key的变化

应用可以观察一个key或者特定范围内的key来监控任何更新。

这是在key `foo` 上进行观察的命令：

```bash
$ etcdctl watch foo
# 在另外一个终端: etcdctl put foo bar
foo
bar
```

这是观察从 `foo` to `foo9` 范围key的命令：

```bash
$ etcdctl watch foo foo9
# 在另外一个终端: etcdctl put foo bar
foo
bar
# 在另外一个终端: etcdctl put foo1 bar1
foo1
bar1
```

## 观察key的历史改动

应用可能想观察etcd中key的历史改动。例如，应用想接收到某个key的所有修改。如果应用一直连接到etcd，那么 `watch` 就足够好了。但是，如果应用或者etcd出错，改动可能发生在出错期间，这样应用就没能实时接收到这个更新。为了保证更新被接收，应用必须能够观察到key的历史变动。为了做到这点，应用可以在观察时指定一个历史修订版本，就像读取key的过往版本一样。

假设我们完成了下列操作序列：

``` bash
etcdctl put foo bar         # revision = 2
etcdctl put foo1 bar1       # revision = 3
etcdctl put foo bar_new     # revision = 4
etcdctl put foo1 bar1_new   # revision = 5
```

这是观察历史改动的例子：

```bash
# 从修订版本 2 开始观察key `foo` 的改动
$ etcdctl watch --rev=2 foo
PUT
foo
bar
PUT
foo
bar_new

# 从修订版本 3 开始观察key `foo` 的改动
$ etcdctl watch --rev=3 foo
PUT
foo
bar_new
```

## 压缩修订版本

如我们提到的，etcd保存修订版本以便应用可以读取key的过往版本。但是，为了避免积累无限数量的历史数据，压缩过往的修订版本就变得很重要。压缩之后，etcd删除历史修订版本，释放资源来提供未来使用。所有修订版本在压缩修订版本之前的被替代的数据将不可访问。

这是压缩修订版本的命令：

```bash
$ etcdctl compact 5
compacted revision 5

# 在压缩修订版本之前的任何修订版本都不可访问
$ etcdctl get --rev=4 foo
Error:  rpc error: code = 11 desc = etcdserver: mvcc: required revision has been compacted
```

## 授予租约

应用可以为etcd集群里面的key授予租约。当key被附加到租约时，它的生存时间被绑定到租约的生存时间，而租约的生存时间相应的被time-to-live (TTL)管理。租约的实际TTL值是不低于最小TTL，由etcd集群选择。一旦租约的TTL到期，租约就过期而且所有附带的key都将被删除。

这是授予租约的命令：

```bash
# 授予租约，TTL为10秒
$ etcdctl lease grant 10
lease 32695410dcc0ca06 granted with TTL(10s)

# 附加key foo到租约32695410dcc0ca06
$ etcdctl put --lease=32695410dcc0ca06 foo bar
OK
```

## 撤销租约

应用通过租约id可以撤销租约。撤销租约将删除所有它附带的key。

假设我们完成了下列的操作：

```bash
$ etcdctl lease grant 10
lease 32695410dcc0ca06 granted with TTL(10s)
$ etcdctl put --lease=32695410dcc0ca06 foo bar
OK
```

这是撤销同一个租约的命令：

```bash
$ etcdctl lease revoke 32695410dcc0ca06
lease 32695410dcc0ca06 revoked

$ etcdctl get foo
# 空应答，因为租约撤销导致foo被删除
```

## 维持租约

应用可以通过刷新key的TTL来维持租约，以便租约不过期。

假设我们完成了下列操作：

```bash
$ etcdctl lease grant 10
lease 32695410dcc0ca06 granted with TTL(10s)
```

这是维持同一个租约的命令：

```bash
$ etcdctl lease keep-alive 32695410dcc0ca0
lease 32695410dcc0ca0 keepalived with TTL(100)
lease 32695410dcc0ca0 keepalived with TTL(100)
lease 32695410dcc0ca0 keepalived with TTL(100)
...
```

> 注： 上面的这个命令，不是单次续约，而是一直维持这个租约。