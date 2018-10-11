# GlusterFS

k8s集群中，怎么也会遇到数据持久化的问题，需要用到PV和PVC，把数据存储到一个固定的节点上。

[`GlusterFS`](https://docs.gluster.org/en/latest/Administrator%20Guide/GlusterFS%20Introduction/) 是一个可伸缩的分布式文件存储方案，Kubernetes 的StorageClass支持的驱动中，推荐  使用GlusterFS

[https://kubernetes.io/docs/concepts/storage/storage-classes/](https://kubernetes.io/docs/concepts/storage/storage-classes/)

![](../.gitbook/assets/image%20%2825%29.png)

## 开始安装 

GlusterFS官方文档：[https://docs.gluster.org/en/latest/Quick-Start-Guide/Quickstart/](https://docs.gluster.org/en/latest/Quick-Start-Guide/Quickstart/)

GlusterFS+Kubernetes官方文档：[https://github.com/gluster/gluster-kubernetes\#quickstart](https://github.com/gluster/gluster-kubernetes#quickstart)

因为文件系统功能简单，使用相对稳定，一次部署，基本上不会再做删减，因此，我们没有必要用Kubernetes去部署GlusterFS，手工部署反而更简单，依赖少。

### 安装前的准备

最少两台服务器，用replication模式，一主一备，防止数据库丢失，高可用。每台服务器2CPU，2G内存，100G SSD硬盘，网络带宽1G以上，安装Ubuntu 16.04

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

### 停止卷

```bash
root@10-103-1-11:~# gluster
gluster> volume stop kube-vol
Stopping volume will make its data inaccessible. Do you want to continue? (y/n) y
volume stop: kube-vol: success
```

### 删除卷

```text
gluster> volume delete kube-vol
Deleting volume will erase all information about the volume. Do you want to continue? (y/n) y
volume delete: kube-vol: success
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

### 添加entrypoint

在gluster服务器上查看服务端口

```text
root@10-103-1-11:~# netstat -nptl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      978/sshd
tcp        0      0 0.0.0.0:49157           0.0.0.0:*               LISTEN      1324/glusterfsd
tcp        0      0 0.0.0.0:24007           0.0.0.0:*               LISTEN      1144/glusterd
tcp6       0      0 :::22                   :::*                    LISTEN      978/sshd
```

我们进程`glusterd`对应的端口`24007`就是服务端口

开始创建entryppint

{% code-tabs %}
{% code-tabs-item title="gluster-endpoints.yaml" %}
```yaml
---
kind: Endpoints
apiVersion: v1
metadata:
  labels:
    app: external-glusterfs
  name: glusterfs
  namespace: common
subsets:
- addresses:
  - ip: 10.103.1.11
  - ip: 10.103.1.119
  ports:
  - port: 24007
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 配置 service

{% code-tabs %}
{% code-tabs-item title="gluster-endpoints.yaml" %}
```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: external-glusterfs
  name: glusterfs
  namespace: common
spec:
  ports:
  - port: 24007
    protocol: TCP
    targetPort: 24007

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 创建测试 pod

`volumes[0].glusterfs.endpoints`对应的值是上面创建`service`的名字

`volumes[0].glusterfs.path`是上面创建的`volume`名字

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: glusterfs-debug
  namespace: common
spec:
  containers:
  - name: glusterfs
    image: nginx
    volumeMounts:
    - mountPath: "/mnt/glusterfs"
      name: glusterfsvol
  volumes:
  - name: glusterfsvol
    glusterfs:
      endpoints: glusterfs
      path: gv0
      readOnly: true
```

查看pod状态，确认启动成功

```text
kubectl describe pods/glusterfs
```

### 安装配置Heketi

上面的方式，适合只有很少PV需求的场景，如果有很多Pod需要很多PV，指向Glusterfs不同的卷（或者说不同的目录），那么每次都要手工创建Glusterfs Volume再创建对应的PV就会很麻烦。因此，我们需要一种自动的方式去生成所需资源。K8s支持自定义PVC的`StorageClass`，通过它和`Heketi`配合，就可以自动创建所需的存储资源，不同`Deployment`可以拥有独立目录。

参考文档：

{% embed url="https://github.com/gluster/gluster-kubernetes/tree/master/docs/examples/dynamic\_provisioning\_external\_gluster" %}

{% embed url="https://github.com/heketi/heketi" %}

添加heketi配置文件：

{% code-tabs %}
{% code-tabs-item title="/etc/heketi/heketi.json" %}
```javascript
{
  "_port_comment": "Heketi Server Port Number",
  "port": "8080",

  "_use_auth": "Enable JWT authorization. Please enable for deployment",
  "use_auth": false,

  "_jwt": "Private keys for access",
  "jwt": {
    "_admin": "Admin has access to all APIs",
    "admin": {
      "key": "My Secret"
    },
    "_user": "User only has access to /volumes endpoint",
    "user": {
      "key": "My Secret"
    }
  },

  "_glusterfs_comment": "GlusterFS Configuration",
  "glusterfs": {

    "_executor_comment": "Execute plugin. Possible choices: mock, ssh",
    "executor": "ssh",
    "sshexec": {
      "keyfile": "/etc/heketi/heketi_key",
      "user": "ubuntu",
      "port": "22",
      "fstab": "/etc/fstab",
      "sudo": true
    },

    "_db_comment": "Database file name",
    "db": "/var/lib/heketi/heketi.db"
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

这里需要注意的`glusterfs.sshexec.sudo`为`true`，否则heketi服务会提示没有权限

**启动`heketi`服务：**为了方便调试，我们在本地用docker启动服务，生产环境，我们可能会在一台独立电脑上启动服务，或者用K8s在Pod中运行这个服务。

```text
docker run --name heketi -v /Users/xiaohui/coding/heketi:/etc/heketi heketi/heketi
```

这里，我们映射了本地目录`/Users/xiaohui/coding/heketi`到容器的`/etc/heketi`

把上面的`heketi.json`文件放到本地目录`/Users/xiaohui/coding/heketi，同时添加`topylogy（Glusterfs Cluster的拓扑图）文件：

{% code-tabs %}
{% code-tabs-item title="/etc/heketi/topology.json" %}
```javascript
{
  "clusters": [
    {
      "nodes": [
        {
          "node": {
            "hostnames": {
              "manage": [
                "10.103.1.11"
              ],
              "storage": [
                "10.103.1.11"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/sdb"
          ]
        },
        {
          "node": {
            "hostnames": {
              "manage": [
                "10.103.1.119"
              ],
              "storage": [
                "10.103.1.119"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/sdb"
          ]
        }
      ]
    }
  ]
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

这个文件的目的是定义Glusterfs服务器地址和磁盘。

然后新开 一个终端，进入heketi容器：

```text
docker exec -it heketi bash
```

当我们手工在Ubuntu上安装Glusterfs之后，用heketi-cli加载集群配置时，可能会出现如下错误：

```text
[root@b5ff52589fc7 ~]# heketi-cli topology load --json /etc/heketi/topology.json
Creating cluster ... ID: ea58edbcd5880a426771e73d0bd5c0df
	Allowing file volumes on cluster.
	Allowing block volumes on cluster.
	Creating node 10.103.1.11 ... Unable to create node: New Node doesn't have glusterd running
	Creating node 10.103.1.119 ... Unable to create node: New Node doesn't have glusterd running
```

查看Heketi HTTP服务器端日志，会发现如下错误：

```text
[negroni] Started GET /queue/a569809c6f54685276f2331aa4891e5f
[negroni] Completed 500 Internal Server Error in 103.1µs
[negroni] Started POST /nodes
[cmdexec] INFO 2018/08/04 10:29:40 Check Glusterd service status in node 10.103.1.11
[cmdexec] DEBUG 2018/08/04 10:29:41 /src/github.com/heketi/heketi/pkg/utils/ssh/ssh.go:174: Host: 10.103.1.11:22 Command: /bin/bash -c 'systemctl status glusterd'
```

这里可以找到原因：`systemctl status glusterd`没有找到服务，是因为Ubuntu安装的gluster服务名是`glusterfs-server`

```text
root@10-103-1-11:~# systemctl status glusterfs-server.service
● glusterfs-server.service - LSB: GlusterFS server
   Loaded: loaded (/etc/init.d/glusterfs-server; bad; vendor preset: enabled)
  Drop-In: /etc/systemd/system/glusterfs-server.service.d
           └─override.conf
   Active: active (running) since Sat 2018-08-04 18:26:14 CST; 6min ago
     Docs: man:systemd-sysv-generator(8)
  Process: 9703 ExecStart=/etc/init.d/glusterfs-server start (code=exited, status=0/SUCCESS)
    Tasks: 40
   Memory: 126.6M
      CPU: 1.775s
```

目前，Heketi写死了服务名称，无法配置 [https://github.com/heketi/heketi/blob/6d80b73a799084cd28776e25857225c645756aaf/executors/cmdexec/peer.go\#L72](https://github.com/heketi/heketi/blob/6d80b73a799084cd28776e25857225c645756aaf/executors/cmdexec/peer.go#L72)

![](../.gitbook/assets/image%20%2827%29.png)

因此，我们只有创建一个systemd别名：`glusterd.service` =&gt; `glusterfs-server.service`

方法如下：

```bash
root@10-103-1-11:~# systemctl edit glusterfs-server.service
```

添加内容：

```text
[Install]
Alias=glusterd.service
```

获取服务的文件地址`/run/systemd/generator.late/glusterfs-server.service`

```text
root@10-103-1-11:~# systemctl cat glusterfs-server.service
# /run/systemd/generator.late/glusterfs-server.service
# Automatically generated by systemd-sysv-generator

[Unit]
Documentation=man:systemd-sysv-generator(8)
SourcePath=/etc/init.d/glusterfs-server
Description=LSB: GlusterFS server
Before=multi-user.target
Before=multi-user.target
Before=multi-user.target
```

创建软连接：

```text
ln -sf /run/systemd/generator.late/glusterfs-server.service /etc/systemd/system/glusterd.service
```

启动服务：

```text
systemctl daemon-reload
systemctl enable glusterd
systemctl start glusterd
```

现在我们再次加载Glusterfs Cluster Topology 文件

```text
heketi-cli topology load --json /etc/heketi/topology.json
[root@c63599638219 /]# heketi-cli topology load --json /etc/heketi/topology.json
	Found node 10.103.1.11 on cluster 2748c838c9e5433c140dd580ce8d92ba
		Adding device /dev/vdb ... OK
	Found node 10.103.1.119 on cluster 2748c838c9e5433c140dd580ce8d92ba
		Adding device /dev/vdb ... OK
```

{% hint style="info" %}
这里需要注意，磁盘/dev/sdb必须是一个空磁盘并且没有被挂载到系统，否则会添加失败。Heketi这么做的目的也是为了保护我们的磁盘数据，万一设置了错误的磁盘，不至于丢失数据。
{% endhint %}

如果磁盘的确已经被挂载或者已经被格式化，我们需要做如下操作：

```text
卸载磁盘
umount /dev/sdb
# 删除掉挂载信息
vi /etc/fstab
# 清除磁盘分区
wipefs /dev/vdb -a
```

#### 测试heketi-cli命令

获取集群

```text
[root@c63599638219 /]# heketi-cli cluster list
Clusters:
Id:2748c838c9e5433c140dd580ce8d92ba [file][block]
```

获取节点列表

```text
[root@c63599638219 /]# heketi-cli node list
Id:783947cc37876e44c1fdee6f1690c1d7	Cluster:2748c838c9e5433c140dd580ce8d92ba
Id:e935a6a52e562477da79d6667401a505	Cluster:2748c838c9e5433c140dd580ce8d92ba
```

创建Volume

```text
[root@c63599638219 /]# heketi-cli volume create --size=10 --replica=2  --name my-vol
Name: my-vol
Size: 10
Volume Id: 66f29b294df96b378548f10ee898eaf5
Cluster Id: 2748c838c9e5433c140dd580ce8d92ba
Mount: 10.103.1.119:my-vol
Mount Options: backup-volfile-servers=10.103.1.11
Block: false
Free Size: 0
Block Volumes: []
Durability Type: replicate
Distributed+Replica: 2
```

{% hint style="info" %}
创建Volume时可能会出现错误“usr sbin thin\_check no such file or directory”，需要在每台Glusterfs服务器上安装：apt install thin-provisioning-tools
{% endhint %}

获取卷列表

```text
[root@c63599638219 /]# heketi-cli volume list
Id:66f29b294df96b378548f10ee898eaf5    Cluster:2748c838c9e5433c140dd580ce8d92ba    Name:my-vol
```

获取拓扑信息

```text
[root@c63599638219 /]# heketi-cli topology info

Cluster Id: 2748c838c9e5433c140dd580ce8d92ba

    File:  true
    Block: true

    Volumes:

	Name: my-vol
	Size: 10
	Id: 66f29b294df96b378548f10ee898eaf5
	Cluster Id: 2748c838c9e5433c140dd580ce8d92ba
	Mount: 10.103.1.119:my-vol
	Mount Options: backup-volfile-servers=10.103.1.11
	Durability Type: replicate
	Replica: 2
	Snapshot: Disabled

		Bricks:
			Id: 44b8c2bdbd04fbeadd197e10a25701b0
			Path: /var/lib/heketi/mounts/vg_26cea42d1a71151288db4cab8477bc86/brick_44b8c2bdbd04fbeadd197e10a25701b0/brick
			Size (GiB): 10
			Node: e935a6a52e562477da79d6667401a505
			Device: 26cea42d1a71151288db4cab8477bc86

			Id: de363568400f13f28e0bb83845c549bd
			Path: /var/lib/heketi/mounts/vg_75a66dbe5b2c64db3678f052837fc9c2/brick_de363568400f13f28e0bb83845c549bd/brick
			Size (GiB): 10
			Node: 783947cc37876e44c1fdee6f1690c1d7
			Device: 75a66dbe5b2c64db3678f052837fc9c2


    Nodes:

	Node Id: 783947cc37876e44c1fdee6f1690c1d7
	State: online
	Cluster Id: 2748c838c9e5433c140dd580ce8d92ba
	Zone: 1
	Management Hostnames: 10.103.1.119
	Storage Hostnames: 10.103.1.119
	Devices:
		Id:75a66dbe5b2c64db3678f052837fc9c2   Name:/dev/vdb            State:online    Size (GiB):99      Used (GiB):10      Free (GiB):89
			Bricks:
				Id:de363568400f13f28e0bb83845c549bd   Size (GiB):10      Path: /var/lib/heketi/mounts/vg_75a66dbe5b2c64db3678f052837fc9c2/brick_de363568400f13f28e0bb83845c549bd/brick

	Node Id: e935a6a52e562477da79d6667401a505
	State: online
	Cluster Id: 2748c838c9e5433c140dd580ce8d92ba
	Zone: 1
	Management Hostnames: 10.103.1.11
	Storage Hostnames: 10.103.1.11
	Devices:
		Id:26cea42d1a71151288db4cab8477bc86   Name:/dev/vdb            State:online    Size (GiB):99      Used (GiB):10      Free (GiB):89
			Bricks:
				Id:44b8c2bdbd04fbeadd197e10a25701b0   Size (GiB):10      Path: /var/lib/heketi/mounts/vg_26cea42d1a71151288db4cab8477bc86/brick_44b8c2bdbd04fbeadd197e10a25701b0/brick
[root@c63599638219 /]#
```

获取磁盘信息

```text
[root@c63599638219 /]# heketi-cli device info 26cea42d1a71151288db4cab8477bc86
Device Id: 26cea42d1a71151288db4cab8477bc86
Name: /dev/vdb
State: online
Size (GiB): 99
Used (GiB): 10
Free (GiB): 89
Bricks:
Id:44b8c2bdbd04fbeadd197e10a25701b0   Size (GiB):10      Path: /var/lib/heketi/mounts/vg_26cea42d1a71151288db4cab8477bc86/brick_44b8c2bdbd04fbeadd197e10a25701b0/brick
```

删除卷

```text
[root@c63599638219 /]# heketi-cli volume delete 66f29b294df96b378548f10ee898eaf5
Volume 66f29b294df96b378548f10ee898eaf5 deleted
```



