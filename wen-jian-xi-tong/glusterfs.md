# GlusterFS

k8s集群中，怎么也会遇到数据持久化的问题，需要用到PV和PVC，把数据存储到一个固定的节点上。

[`GlusterFS`](https://docs.gluster.org/en/latest/Administrator%20Guide/GlusterFS%20Introduction/) 是一个可伸缩的分布式文件存储方案，Kubernetes 的StorageClass支持的驱动中，推荐  使用GlusterFS

[https://kubernetes.io/docs/concepts/storage/storage-classes/](https://kubernetes.io/docs/concepts/storage/storage-classes/)

![](../.gitbook/assets/image%20%2822%29.png)

## 开始安装 

GlusterFS官方文档：[https://docs.gluster.org/en/latest/Quick-Start-Guide/Quickstart/](https://docs.gluster.org/en/latest/Quick-Start-Guide/Quickstart/)

GlusterFS+Kubernetes官方文档：[https://github.com/gluster/gluster-kubernetes\#quickstart](https://github.com/gluster/gluster-kubernetes#quickstart)

因为文件系统功能简单，使用相对稳定，一次部署，基本上不会再做删减，因此，我们没有必要用Kubernetes去部署GlusterFS，手工部署反而更简单，依赖少。

### 安装前的准备

最少两台服务器，用replication模式，一主一备，防止数据库丢失，高可用。每台服务器2CPU，2G内存，100G SSD硬盘，网络带宽1G以上，安装Ubuntu 14.04

### 安装

```bash
sudo apt-get install software-properties-common
sudo apt-get update
sudo apt-get install glusterfs-server -y
```

## 配置

我们有两台服务器：`10.103.1.11`和`10.103.1.119`，在`10.103.1.11`上执行命令，连接`10.103.1.119`，组成集群

```bash
sudo gluster peer probe 10.103.1.119
sudo gluster peer status

Number of Peers: 1

Hostname: 10.103.1.119
Uuid: 60fedfe2-cb44-41f7-9b3d-cf2901793a85
State: Peer in Cluster (Connected)
```

格式化两台服务器的数据盘，自动挂载

```bash
mkfs.xfs -i size=512 -f /dev/vdb
# 设置开机自动挂载磁盘
echo "/dev/vdb /export/vdb xfs defaults 0 0"  >> /etc/fstab
# 创建目录
mkdir -p /export/vdb && mount -a && mkdir -p /export/vdb/brick1
```

### 添加第一个卷

在`10.103.1.11`上执行

```bash
gluster volume create gv0 replica 2 10.103.1.119:/export/vdb/brick1 10.103.1.11:/export/vdb/brick1
```

`gluster volume info`查看券状态

```text
root@10-103-1-11:~# gluster volume info

Volume Name: gv0
Type: Replicate
Volume ID: 5faab39a-c2d4-47b8-a612-29f6d53616cf
Status: Created
Snapshot Count: 0
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: 10.103.1.119:/export/vdb/brick1
Brick2: 10.103.1.11:/export/vdb/brick1
Options Reconfigured:
transport.address-family: inet
performance.readdir-ahead: on
nfs.disable: on
```

启动卷：

```text
root@10-103-1-11:~# gluster volume start gv0
volume start: gv0: success
root@10-103-1-11:~# gluster volume info

Volume Name: gv0
Type: Replicate
Volume ID: 5faab39a-c2d4-47b8-a612-29f6d53616cf
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: 10.103.1.119:/export/vdb/brick1
Brick2: 10.103.1.11:/export/vdb/brick1
Options Reconfigured:
transport.address-family: inet
performance.readdir-ahead: on
nfs.disable: on
```

## 客户端挂载卷

 卷可以像普通nfs一样通过mount挂载，只需要安装glusterfs支持即可。

这里需要一天新的客户端电脑，安装`glusterfs`客户端：

```bash
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:gluster/glusterfs-3.8
sudo apt-get update
sudo apt-get install glusterfs-client
```

### 挂载卷

```bash
sudo mkdir /mnt/my-vol
sudo mount -t glusterfs 10.103.1.11:/gv0/dir1 /mnt/my-vol
df -h /mnt/my-vol

root@10-10-61-207:~# df -h /mnt/my-vol
Filesystem        Size  Used Avail Use% Mounted on
10.103.1.11:/gv0  100G   33M  100G   1% /mnt/my-vol
```

## 开启磁盘配额

在服务端开启配额

```bash
gluster volume quota gv0 enable
```

在服务端设置一个目录的磁盘配额

```bash
gluster volume quota gv0 limit-usage / 10GB
```

客户端查看配额

```bash
root@10-10-61-207:~# df -h /mnt/my-vol
Filesystem        Size  Used Avail Use% Mounted on
10.103.1.11:/gv0   20G     0   20G   0% /mnt/my-vol
```

## Kubernetes使用GlusterFS

K8s官方文档[https://github.com/kubernetes/examples/tree/master/staging/volumes/glusterfs](https://github.com/kubernetes/examples/tree/master/staging/volumes/glusterfs)



