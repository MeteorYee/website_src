+++
title = "Zookeeper"
date = "2024-04-20"
+++

Tens of thousands of the applications are running on distributed systems today, where ZooKeeper plays an important role in order to make the whole system work consistently. Here are my notes on it.

## What is ZK
ZK provides the uses a consistent view of a distributed system, and building blocks for synchronization across a cluster of nodes. For example, it has the ability to save the meta information, such as membership configurations, of a cluster. What’s more, users can use it to implement a distributed lock required by upper level applications.

### Data Model
ZK exposes a file system like structure to the users. Each ‘directory’ is a ‘znode’ in terms of ZK. There are two types of znode: regular and ephemeral. A regular znode is created and deleted by the clients, whereas an ephemeral one is created by the clients but automatically deleted when they normally exit or accidentally terminated.

Its APIs are really similar to the ones of a file system, from [1]:

```
create(path, data, flags);
delete(path, version);
exists(path, watch);
getData(path, watch);
setData(path, data, version);
getChildren(path, watch);
sync(path);
```

Specifically, the watch parameter of the APIs means an interesting watch mechanism provided by ZK. To illustrate, the clients can watch some designated znodes. When there are changes happen on them, those who are concerned will receive the notifications and act accordingly.

### Architecture
ZK has a relaxed consistency guarantee or in its word: A-linearizable. A read replica may not get the latest value immediately. However, if it watches on some znode, it will get a notification that there are changes. Then it’s guaranteed to see the value changed at least by this time, as there could be more changes coming up.

According to [2], ZK has a primary-backup architecture.

![zk overview](/images/tech/zk_arch.svg)

Zab
todo…

todo: Zab vs Raft

## Reference
[1]. P. Hunt, M. Konar, F. P. Junqueira, and B. Reed, “ZooKeeper: Wait-free coordination for Internet-scale systems,” in USENIX ATC’10: Proceedings of the 2010 USENIX Annual Technical Conference. USENIX Association, 2010.

[2]. JUNQUEIRA, F. P., REED, B. C., AND SERAFINI, M. Zab: High-performance broadcast for primary-backup systems. In Proc. DSN’11, IEEE/IFIP Int’l Conf. on Dependable Systems & Networks (2011), IEEE Computer Society, pp. 245–256.