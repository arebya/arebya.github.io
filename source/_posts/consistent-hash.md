---
title: consistent_hash
date: 2018-03-28 15:13:37
tags: 分布式
---

### 为什么需要一致性hash


### 算法的理解

http://en.wikipedia.org/wiki/Consistent_hashing

### 具体实现及应用

Dubbo中一致性Hash的实现：com.alibaba.dubbo.rpc.cluster.loadbalance.ConsistentHashLoadBalance

Motan中一致性Hash的实现：com.weibo.api.motan.cluster.loadbalance.ConsistentHashLoadBalance<T>

Guava中一致性Hash实现：com.google.common.hash.Hashing.consistentHash(*)
