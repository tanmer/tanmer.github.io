---
description: '支持Sharding, Replication的大数据用户数据库'
---

# Postgre XL

官网：[https://www.postgres-xl.org/](https://www.postgres-xl.org/)

参考：[https://www.jianshu.com/p/82aaf352b772](https://www.jianshu.com/p/82aaf352b772)

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

## 安装

这里有3个角色概念：

1. GTM
2. Coordinator
3. Datanode

官方推荐Coordinator和Datanode在同一节点，GTM至少有一个Standby，因此，我们安装一套初始环境，节点做如下安排：

Node1：GTM =&gt; 2CPUs + 4G MEM + 20G DISK

Node2：GTM Standby =&gt; 2CPUs + 4G MEM + 20G DISK

Node3: coord\_1 + datanode\_1  =&gt; 2CPUs + 4G MEM + 100G SSD

Node4: coord\_2 + datanode\_2  =&gt; 2CPUs + 4G MEM + 100G SSD

Node5: datanode\_1\_slave =&gt; 2CPUs + 4G MEM + 100G SSD

Node6: datanode\_2\_slave =&gt; 2CPUs + 4G MEM + 100G SSD

6个节点，2个GTMs，2个Shards, 每个Shard有一个Slave

### 下载、编译

在每个节点编译安装Postgres-XL，版本为XL9\_5\_R1\_6

```bash
sudo apt-get install build-essential libreadline-dev zlib1g-dev bison flex -y
git clone git://git.postgresql.org/git/postgres-xl.git
cd postgres-xl
git checkout XL9_5_R1_6
./configure
make -j 2
sudo make install
cd contrib
make -j 2
sudo make install
```

编辑`/etc/environment`，添加`/usr/local/pgsql/bin/`

在Node1上使用`pgxc_ctl`工具配置系统

```bash
pgxc_ctl
# 进入PGXC控制台
PGXC prepare config empty
PGXC exit
```

{% code-tabs %}
{% code-tabs-item title="输出：" %}
```text
ubuntu@10-103-0-59:~/pgxc_ctl$ pgxc_ctl
/bin/bash
Installing pgxc_ctl_bash script as /home/ubuntu/pgxc_ctl/pgxc_ctl_bash.
Installing pgxc_ctl_bash script as /home/ubuntu/pgxc_ctl/pgxc_ctl_bash.
Reading configuration using /home/ubuntu/pgxc_ctl/pgxc_ctl_bash --home /home/ubuntu/pgxc_ctl --configuration /home/ubuntu/pgxc_ctl/pgxc_ctl.conf
Finished reading configuration.
   ******** PGXC_CTL START ***************

Current directory: /home/ubuntu/pgxc_ctl
PGXC prepare config empty
PGXC exit
```
{% endcode-tabs-item %}
{% endcode-tabs %}

修改`~/pgxc_ctl/pgxc_ctl.conf`文件，这里主要修改`pgxcOwner=postgres`和`dataDirRoot=/data/pgxl/nodes`

再次运行`pgxc_ctl`，开始添加集群角色





