# 配置标记

> 注： 内容翻译自 [Configuration flags](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/configuration.md)

etcd可以通过命令行标记和环境变量来配置。命令行上设置的选项优先于环境变量。

对于标记 `--my-flag` 环境变量的格式是 `ETCD_MY_FLAG`。 适用于所有标记。

[正式的ectd端口](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=etcd) 是2379用于客户端连接，而2380用于伙伴通讯。etcd端口可以设置为接受TLS通讯，non-TLS通讯，或者同时有 TLS 和 non-TLS 通讯。

为了在 linux 启动试使用自定义设置自动启动 etcd ，强烈推荐使用 [systemd](http://freedesktop.org/wiki/Software/systemd/)单元。

## 成员标记

### --name

+ 这个成员的可读性的名字.
+ 默认: "default"
+ 环境变量: ETCD_NAME
+ 这个值被作为这个节点自己的入口中被引用， 在 `--initial-cluster` 标记(例如, `default=http://localhost:2380`)中列出。如果使用 [static bootstrapping](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/clustering.md#static),这需要匹配在标记中使用的key。当使用发现时，每个成员必须有唯一名字。`Hostname` 或 `machine-id` 可以是一个好选择。

### --data-dir

+ 到数据目录的路径.
+ 默认: "${name}.etcd"
+ 环境变量: ETCD_DATA_DIR

### --wal-dir

+ 到专用的wal目录的路径。如果这个标记被设置，etcd将写 WAL 文件到 walDIR 而不是 dataDIR。这容许使用专门的硬盘，并帮助避免日志和其他IO操作之间的IO竞争。
+ 默认: ""
+ 环境变量: ETCD_WAL_DIR

### --snapshot-count

+ 触发快照到硬盘的已提交事务的数量.
+ 默认: "10000"
+ 环境变量: ETCD_SNAPSHOT_COUNT

### --heartbeat-interval

+ 心跳间隔时间 (单位 毫秒).
+ 默认: "100"
+ 环境变量: ETCD_HEARTBEAT_INTERVAL

### --election-timeout

+ 选举的超时时间(单位 毫秒). 阅读 [Documentation/tuning.md](https://github.com/coreos/etcd/blob/master/Documentation/tuning.md#time-parameters) 得到更多详情.
+ 默认: "1000"
+ 环境变量: ETCD_ELECTION_TIMEOUT

### --listen-peer-urls

+ 用于监听伙伴通讯的URL列表。这个标记告诉 etcd 在特定的scheme://IP:port 组合上从它的伙伴接收进来的请求。scheme可是http或者https。如果IP被指定为0.0.0.0,etcd在所有接口上监听给定端口。如果给定IP地址和端口，etcd将监听在给定端口和接口上。多个URL可以用来指定多个地址和端口来监听。etcd将从任何列出来的地址和端口上应答请求。
+ 默认: "http://localhost:2380"
+ 环境变量: ETCD_LISTEN_PEER_URLS
+ 例子: "http://10.0.0.1:2380"
+ 无效例子: "http://example.com:2380" (对于绑定域名无效)

### --listen-client-urls

- 用于监听客户端通讯的URL列表。这个标记告诉 etcd 在特定的scheme://IP:port 组合上从客户端接收进来的请求。scheme可是http或者https。如果IP被指定为0.0.0.0,etcd在所有接口上监听给定端口。如果给定IP地址和端口，etcd将监听在给定端口和接口上。多个URL可以用来指定多个地址和端口来监听。etcd将从任何列出来的地址和端口上应答请求。
+ 默认: "http://localhost:2379"
+ 环境变量: ETCD_LISTEN_CLIENT_URLS
+ 例子: "http://10.0.0.1:2379"
+ 无效例子: "http://example.com:2379" (对于绑定域名无效)

### --max-snapshots

+ 保持的快照文件的最大数量 (0 表示不限制)
+ 默认: 5
+ 环境变量: ETCD_MAX_SNAPSHOTS
+ 对于windows用户默认不限制，而且推荐手工降低到5（或者某些安全偏好)。

### --max-wals

+ 保持的wal文件的最大数量 (0 表示不限制)
+ 默认: 5
+ 环境变量: ETCD_MAX_WALS
+ 对于windows用户默认不限制，而且推荐手工降低到5（或者某些安全偏好)。

### --cors

+ 逗号分割的白名单 origins for CORS (cross-origin resource sharing).
+ 默认: none
+ 环境变量: ETCD_CORS

## 集群标记

`--initial` prefix flags are used in bootstrapping ([static bootstrap][build-cluster], [discovery-service bootstrap][discovery] or [runtime reconfiguration][reconfig]) a new member, and ignored when restarting an existing member.

`--discovery` prefix flags need to be set when using [discovery service][discovery].

### --initial-advertise-peer-urls

+ List of this member's peer URLs to advertise to the rest of the cluster. These addresses are used for communicating etcd data around the cluster. At least one must be routable to all cluster members. These URLs can contain domain names.
+ 默认: "http://localhost:2380"
+ 环境变量: ETCD_INITIAL_ADVERTISE_PEER_URLS
+ example: "http://example.com:2380, http://10.0.0.1:2380"

### --initial-cluster
+ Initial cluster configuration for bootstrapping.
+ 默认: "default=http://localhost:2380"
+ 环境变量: ETCD_INITIAL_CLUSTER
+ The key is the value of the `--name` flag for each node provided. The default uses `default` for the key because this is the default for the `--name` flag.

### --initial-cluster-state
+ Initial cluster state ("new" or "existing"). Set to `new` for all members present during initial static or DNS bootstrapping. If this option is set to `existing`, etcd will attempt to join the existing cluster. If the wrong value is set, etcd will attempt to start but fail safely.
+ 默认: "new"
+ 环境变量: ETCD_INITIAL_CLUSTER_STATE

[static bootstrap]: clustering.md#static

### --initial-cluster-token
+ Initial cluster token for the etcd cluster during bootstrap.
+ 默认: "etcd-cluster"
+ 环境变量: ETCD_INITIAL_CLUSTER_TOKEN

### --advertise-client-urls
+ List of this member's client URLs to advertise to the rest of the cluster. These URLs can contain domain names.
+ 默认: "http://localhost:2379"
+ 环境变量: ETCD_ADVERTISE_CLIENT_URLS
+ example: "http://example.com:2379, http://10.0.0.1:2379"
+ Be careful if advertising URLs such as http://localhost:2379 from a cluster member and are using the proxy feature of etcd. This will cause loops, because the proxy will be forwarding requests to itself until its resources (memory, file descriptors) are eventually depleted.

### --discovery
+ Discovery URL used to bootstrap the cluster.
+ 默认: none
+ 环境变量: ETCD_DISCOVERY

### --discovery-srv
+ DNS srv domain used to bootstrap the cluster.
+ 默认: none
+ 环境变量: ETCD_DISCOVERY_SRV

### --discovery-fallback
+ Expected behavior ("exit" or "proxy") when discovery services fails. "proxy" supports v2 API only.
+ 默认: "proxy"
+ 环境变量: ETCD_DISCOVERY_FALLBACK

### --discovery-proxy
+ HTTP proxy to use for traffic to discovery service.
+ 默认: none
+ 环境变量: ETCD_DISCOVERY_PROXY

### --strict-reconfig-check
+ Reject reconfiguration requests that would cause quorum loss.
+ 默认: false
+ 环境变量: ETCD_STRICT_RECONFIG_CHECK

### --auto-compaction-retention
+ Auto compaction retention for mvcc key value store in hour. 0 means disable auto compaction.
+ 默认: 0
+ 环境变量: ETCD_AUTO_COMPACTION_RETENTION

## Proxy flags

`--proxy` prefix flags configures etcd to run in [proxy mode][proxy]. "proxy" supports v2 API only.

### --proxy
+ Proxy mode setting ("off", "readonly" or "on").
+ 默认: "off"
+ 环境变量: ETCD_PROXY

### --proxy-failure-wait
+ Time (in milliseconds) an endpoint will be held in a failed state before being reconsidered for proxied requests.
+ 默认: 5000
+ 环境变量: ETCD_PROXY_FAILURE_WAIT

### --proxy-refresh-interval
+ Time (in milliseconds) of the endpoints refresh interval.
+ 默认: 30000
+ 环境变量: ETCD_PROXY_REFRESH_INTERVAL

### --proxy-dial-timeout
+ Time (in milliseconds) for a dial to timeout or 0 to disable the timeout
+ 默认: 1000
+ 环境变量 ETCD_PROXY_DIAL_TIMEOUT

### --proxy-write-timeout
+ Time (in milliseconds) for a write to timeout or 0 to disable the timeout.
+ 默认: 5000
+ 环境变量: ETCD_PROXY_WRITE_TIMEOUT

### --proxy-read-timeout
+ Time (in milliseconds) for a read to timeout or 0 to disable the timeout.
+ Don't change this value if using watches because use long polling requests.
+ 默认: 0
+ 环境变量: ETCD_PROXY_READ_TIMEOUT

## Security flags

The security flags help to [build a secure etcd cluster][security].

### --ca-file [DEPRECATED]
+ Path to the client server TLS CA file. `--ca-file ca.crt` could be replaced by `--trusted-ca-file ca.crt --client-cert-auth` and etcd will perform the same.
+ 默认: none
+ 环境变量: ETCD_CA_FILE

### --cert-file
+ Path to the client server TLS cert file.
+ 默认: none
+ 环境变量: ETCD_CERT_FILE

### --key-file
+ Path to the client server TLS key file.
+ 默认: none
+ 环境变量: ETCD_KEY_FILE

### --client-cert-auth
+ Enable client cert authentication.
+ 默认: false
+ 环境变量: ETCD_CLIENT_CERT_AUTH

### --trusted-ca-file
+ Path to the client server TLS trusted CA key file.
+ 默认: none
+ 环境变量: ETCD_TRUSTED_CA_FILE

### --auto-tls
+ Client TLS using generated certificates
+ 默认: false
+ 环境变量: ETCD_AUTO_TLS

### --peer-ca-file [DEPRECATED]
+ Path to the peer server TLS CA file. `--peer-ca-file ca.crt` could be replaced by `--peer-trusted-ca-file ca.crt --peer-client-cert-auth` and etcd will perform the same.
+ 默认: none
+ 环境变量: ETCD_PEER_CA_FILE

### --peer-cert-file
+ Path to the peer server TLS cert file.
+ 默认: none
+ 环境变量: ETCD_PEER_CERT_FILE

### --peer-key-file
+ Path to the peer server TLS key file.
+ 默认: none
+ 环境变量: ETCD_PEER_KEY_FILE

### --peer-client-cert-auth
+ Enable peer client cert authentication.
+ 默认: false
+ 环境变量: ETCD_PEER_CLIENT_CERT_AUTH

### --peer-trusted-ca-file
+ Path to the peer server TLS trusted CA file.
+ 默认: none
+ 环境变量: ETCD_PEER_TRUSTED_CA_FILE

### --peer-auto-tls
+ Peer TLS using generated certificates
+ 默认: false
+ 环境变量: ETCD_PEER_AUTO_TLS

## Logging flags

### --debug
+ Drop the default log level to DEBUG for all subpackages.
+ 默认: false (INFO for all packages)
+ 环境变量: ETCD_DEBUG

### --log-package-levels
+ Set individual etcd subpackages to specific log levels. An example being `etcdserver=WARNING,security=DEBUG` 
+ 默认: none (INFO for all packages)
+ 环境变量: ETCD_LOG_PACKAGE_LEVELS


## Unsafe flags

Please be CAUTIOUS when using unsafe flags because it will break the guarantees given by the consensus protocol.
For example, it may panic if other members in the cluster are still alive.
Follow the instructions when using these flags.

### --force-new-cluster

+ Force to create a new one-member cluster. It commits configuration changes forcing to remove all existing members in the cluster and add itself. It needs to be set to [restore a backup][restore].
+ 默认: false
+ 环境变量: ETCD_FORCE_NEW_CLUSTER

## 其他标记

### --version
+ 打印版本并退出.
+ 默认: false

### --config-file

+ 从文件中装载服务器配置.
+ 默认: none

## 分析标记

### --enable-pprof

+ 通过HTTP服务器开启运行时分析数据。地址是 client URL + "/debug/pprof/"
+ 默认: false

