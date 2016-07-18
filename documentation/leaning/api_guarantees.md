# KV API 保证

> 注： 内容翻译自 [KV API grarentees](https://github.com/coreos/etcd/blob/master/Documentation/leaning/api_guarantees.md)
>
> 注2： 在官网文档页面中没有看到这个文档的链接，我是在git仓库的leaning目录下找到的。

etcd是一致而持久的键值存储，带有微事务(mini-transaction)。键值对存储通过KV API暴露。etcd力图为分布式系统获取最强的一致性和持久性保证。这份规范列举etcd实现的 KV API 保证。

### 考虑的 APIs

* 读 APIs
    * range
    * watch
* 写 APIs
    * put
    * delete
* 联合 (读-改-写) APIs
    * txn

### etcd 明确定义

#### 操作完成

当etcd操作通过一致同意而提交时，视为操作完成，并且因此"执行过" -- 永久保存  -- 被etcd存储引擎。当客户端从etcd服务器接收到应答时，客户端得知操作已经完成。注意，如果客户端和etcd成员之间有超时或者网络中断，客户端可能不确定操作的状态。当有leader选举时，etcd也可能中止操作。在这个事件中，etcd不会发送 `abort` 应答给客户端的未完成的请求。

#### revision/修订版本

修改键值存储的etcd操作被分配有一个唯一的增加的修订版本。事务操作可能多次修改键值存储，但是只分配一个修订版本。被操作修改的键值对的revision属性和操作的revision有同样的值。修订版本可以用来给键值存储做逻辑时间。有较大修订版本的键值对在有较小修订版本的键值对之后修改。两个拥有相同修订版本的键值对是被一个操作同时修改。

### 提供的保证

#### 原子性

所有API请求都是原子的; 操作或者完整的完成，或者完全不。对于观察请求，一个操作生成的所有事件将在一个观察应答中。观察绝不会看到一个操作的部分事件。

#### 一致性

所有API调用确保 [顺序一致性](https://en.wikipedia.org/wiki/Consistency_model#Sequential_consistency), 分布式系统可用的最强的一致性保证。不管客户端的请求发往哪个etcd成员
All API calls ensure [sequential consistency][seq_consistency], the strongest consistency guarantee available from distributed systems. No matter which etcd member server a client makes requests to, a client reads the same events in the same order. If two members complete the same number of operations, the state of the two members is consistent.

For watch operations, etcd guarantees to return the same value for the same key across all members for the same revision. For range operations, etcd has a similar guarantee for [linearized][Linearizability] access; serialized access may be behind the quorum state, so that the later revision is not yet available.

As with all distributed systems, it is impossible for etcd to ensure [strict consistency][strict_consistency]. etcd does not guarantee that it will return to a read the “most recent” value (as measured by a wall clock when a request is completed) available on any cluster member.

#### Isolation

etcd ensures [serializable isolation][serializable_isolation], which is the highest isolation level available in distributed systems. Read operations will never observe any intermediate data.

#### Durability

Any completed operations are durable. All accessible data is also durable data. A read will never return data that has not been made durable.

#### Linearizability

Linearizability (also known as Atomic Consistency or External Consistency) is a consistency level between strict consistency and sequential consistency. 

For linearizability, suppose each operation receives a timestamp from a loosely synchronized global clock. Operations are linearized if and only if they always complete as though they were executed in a sequential order and each operation appears to complete in the order specified by the program. Likewise, if an operation’s timestamp precedes another, that operation must also precede the other operation in the sequence.

For example, consider a client completing a write at time point 1 (*t1*). A client issuing a read at *t2* (for *t2* > *t1*) should receive a value at least as recent as the previous write, completed at *t1*. However, the read might actually complete only by *t3*, and the returned value, current at *t2* when the read began, might be "stale" by *t3*.

etcd does not ensure linearizability for watch operations. Users are expected to verify the revision of watch responses to ensure correct ordering.

etcd ensures linearizability for all other operations by default. Linearizability comes with a cost, however, because linearized requests must go through the Raft consensus process. To obtain lower latencies and higher throughput for read requests, clients can configure a request’s consistency mode to `serializable`, which may access stale data with respect to quorum, but removes the performance penalty of linearized accesses' reliance on live consensus.

[seq_consistency]: https://en.wikipedia.org/wiki/Consistency_model#Sequential_consistency
[strict_consistency]: https://en.wikipedia.org/wiki/Consistency_model#Strict_consistency
[serializable_isolation]: https://en.wikipedia.org/wiki/Isolation_(database_systems)#Serializable
[Linearizability]: #Linearizability
