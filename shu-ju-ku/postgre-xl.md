---
description: '支持Sharding, Replication的大数据用户数据库'
---

# Postgre XL

官网：[https://www.postgres-xl.org/](https://www.postgres-xl.org/)

官网部署文档：[https://www.postgres-xl.org/documentation/server-start.html](https://www.postgres-xl.org/documentation/server-start.html)

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



![](../.gitbook/assets/image%20%2828%29.png)

目前安装方式只支持二进制安装，rpm或deb等格式以后会支持的

### 组件构成

#### GTM

#### GTM-Proxy

#### Coordinator

#### Datanode

## 安装

这里有3个角色概念：

1. GTM
2. Coordinator
3. Datanode

官方推荐Coordinator和Datanode在同一节点，GTM至少有一个Standby，因此，我们安装一套初始环境，节点做如下安排：

Node1：GTM =&gt; 2CPUs + 4G MEM + 20G DISK

Node2：GTM Standby =&gt; 2CPUs + 4G MEM + 20G DISK

Node3: coord\_1 + datanode\_1  =&gt; 2CPUs + 4G MEM + 200G SSD

Node4: coord\_2 + datanode\_2  =&gt; 2CPUs + 4G MEM + 200G SSD

Node5: datanode\_1\_slave =&gt; 2CPUs + 4G MEM + 200G SSD

Node6: datanode\_2\_slave =&gt; 2CPUs + 4G MEM + 200G SSD

6个节点，2个GTMs，2个Shards, 每个Shard有一个Slave

### 创建postgres用户账号

在每个节点，创建postgres账号，创建存放数据的基础目录

```bash
sudo adduser postgres --disabled-password
sudo mkdir -p /data/pgxc/nodes
sudo chown postgres.postgres /data/pgxc/nodes
```

实现从node1向其他节点ssh免登陆

Node1上：

```bash
sudo su postgres
ssh-keygen
```

一路默认回车，会得到两个文件`/home/postgres/.ssh/id_rsa`和`/home/postgres/.ssh/id_rsa.pub`，把`pub`文件复制到`authorized_keys`中

```bash
cp /home/postgres/.ssh/id_rsa.pub /home/postgres/.ssh/authorized_keys
```

登录Node2-Node6，那`/home/postgres/.ssh/authorized_keys`文件的内容复制到每个节点的`/home/postgres/.ssh/authorized_keys`

从Node1用postgres账号登录Node2-6，确认每个节点都不需要输入密码

### 下载、编译

在每个节点编译安装Postgres-XL，版本为XL9\_5\_R1\_6

```bash
sudo apt-get update
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

编辑`/etc/environment`，添加`/usr/local/pgsql/bin`

### 使用pgxc\_ctl工具初始化集群

在Node1上用postgres账号使用`pgxc_ctl`

```bash
sudo su postgres
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

创建`~/pgxc_ctl/pg_hba_extra.conf`文件，这个文件至关重要，把每台服务器的IP都加入信任列表，让每个账户访问数据库都不需要输入密码。否则，在后面的步骤中，创建的新用户将无法访问数据库，即使密码输入正确也不行。

{% code-tabs %}
{% code-tabs-item title="pg\_hba\_extra.conf" %}
```text
host all all 10.103.0.59/32 trust
host all all 10.103.0.202/32 trust
host all all 10.103.0.97/32 trust
# 其他IP需要输入密码
host all all 0.0.0.0/0 md5
```
{% endcode-tabs-item %}
{% endcode-tabs %}

再次运行`pgxc_ctl`，开始添加集群角色

```bash
# 注：PGXC命令行无法所以用变量
# node1=10.103.0.59
# node3=10.103.0.202
# node5=10.103.0.97
# 添加GTM
pgxc_ctl
PGXC add gtm master gtm1 10.103.0.59 6666 /data/pgxc/nodes/gtm
# 检查GTM状态，这时应该显示的 Running: gtm master
PGXC monitor all
# 添加coordinator
PGXC add coordinator master coord1 10.103.0.202 5432 5433 /data/pgxc/nodes/coord_master none none
PGXC monitor all
# Running: gtm master
# Running: coordinator master coord1
# 添加datanode
PGXC add datanode master dn1 10.103.0.202 5434 5435 /data/pgxc/nodes/datanode none none none
# Running: gtm master
# Running: coordinator master coord1
# Running: datanode master dn1
PGXC add coordinator master coord2 10.103.0.97 5432 5433 /data/pgxc/nodes/coordinator none none
PGXC add datanode master dn2 10.103.0.97 5434 5435 /data/pgxc/nodes/datanode none none none
PGXC monitor all
# Running: gtm master
# Running: coordinator master coord1
# Running: coordinator master coord2
# Running: datanode master dn1
# Running: datanode master dn2
```

现在，1个GTM，2个Coordinator，2个datanode组成了集群

在node1上测试一下：

```text
psql -h 10.103.0.202

postgres=# create database test1;
CREATE DATABASE
postgres=# \c test1
You are now connected to database "test1" as user "postgres".
test1=# SELECT * FROM pgxc_node;
 node_name | node_type | node_port |  node_host   | nodeis_primary | nodeis_preferred |   node_id
-----------+-----------+-----------+--------------+----------------+------------------+-------------
 coord1    | C         |      5432 | 10.103.0.202 | f              | f                |  1885696643
 dn1       | D         |      5434 | 10.103.0.202 | f              | f                |  -560021589
 coord2    | C         |      5432 | 10.103.0.97  | f              | f                | -1197102633
 dn2       | D         |      5434 | 10.103.0.97  | f              | f                |   352366662
(4 rows)
```

### 添加GTM

```text
PGXC add gtm master gtm1 10.103.0.59 6666 /data/pgxc/nodes/gtm
```

### 添加Coordinator

```text
PGXC add coordinator master coord1 10.103.0.202 5432 5433 /data/pgxc/nodes/coordinator none none
```

### 添加Datanode

Master:

```text
PGXC add datanode master dn1 10.103.0.97 5434 5435 /data/pgxc/nodes/datanode_master none none none
```

Slave:

```text
PGXC add datanode slave dn1 10.103.0.202 5434 5435 /data/pgxc/nodes/datanode_slave none /data/pgxc/nodes/datanode_archlog
```

{% hint style="info" %}
注意：添加slave时，名称，端口都要和master一样
{% endhint %}



