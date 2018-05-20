---
description: '支持Sharding, Replication的大数据用户数据库'
---

# Postgre XL

官网：[https://www.postgres-xl.org/](https://www.postgres-xl.org/)

## 概述

Postgres XL，XL是eXtensible Lattice的简写，意识是可扩展的盒子。

### 主要特点：

#### 可伸缩（Scalable）

Postgres XL可实现数据分表存储在多个节点中，或在多节点中同步副本集

#### 完全支持ACID（Fully ACID）

* Atomicity （原子性，或称不可分割性）
* Consistency（一致性）
* Isolation（隔离性，或称独立性）
* Durability（持续性）



![](../.gitbook/assets/image%20%2823%29.png)

目前安装方式只支持二进制安装，rpm或deb等格式以后会支持的

