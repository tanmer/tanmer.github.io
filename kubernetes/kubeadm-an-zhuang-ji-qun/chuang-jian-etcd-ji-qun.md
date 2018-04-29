# 创建etcd集群

官方文档：[https://kubernetes.io/docs/setup/independent/high-availability/\#before-you-begin](https://kubernetes.io/docs/setup/independent/high-availability/#before-you-begin)

Kubernetes的数据库是`etcd`，要让k8s实现高可用，首先就需要实现etcd的高可用，因此本文介绍如何创建etcd集群

`etcd`集群需要奇数个节点才能选举出`leader`，因此我们至少需要`3`个节点来运行`etcd`

## 安装etcd

因为我们初始化k8s集群时，第一个`master`已经安装来`etcd`，比如这个master节点名叫`node1`，所以我们还需要两个节点：`node2`, `node3`

安装etcd之前，我们需要先在`node1`上检查etcd的版本：

```bash
root@10-9-126-15:~# grep image /etc/kubernetes/manifests/etcd.yaml
    image: k8s.gcr.io/etcd-amd64:3.1.12
```

确定版本为`3.1.12`

进入`etcd`下载页面：[https://github.com/coreos/etcd/releases](https://github.com/coreos/etcd/releases)，根据Linux安装脚本，开始在`node2`和`node3`上面安装

把每个节点的IP地址记录在变量中，后面会用到：

```text
node1_ip=10.9.126.15
node2_ip=10.9.65.99
node3_ip=10.9.28.242
```

```bash
ETCD_VER=v3.1.12
GITHUB_URL=https://github.com/coreos/etcd/releases/download
DOWNLOAD_URL=${GITHUB_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

/tmp/etcd-download-test/etcd --version
<<COMMENT
etcd Version: 3.1.12
Git SHA: 918698add
Go Version: go1.8.7
Go OS/Arch: linux/amd64
COMMENT

ETCDCTL_API=3 /tmp/etcd-download-test/etcdctl version
<<COMMENT
etcdctl version: 3.1.12
API version: 3.1
COMMENT
```

为了简化安装流程，我们先用`apt`安装`etcd`，然后把我们指定版本的etcd二进制文件覆盖apt安装的文件：

```bash
apt-get update
apt-get install etcd -y
```

把之前下载额二进制文件覆盖apt安装的文件：

```bash
systemctl stop etcd
mv /tmp/etcd-download-test/etcd /usr/bin/
mv /tmp/etcd-download-test/etcdctl /usr/bin/
systemctl start etcd
systemctl status etcd
```

在`node2`和`node3`上修改配置文件：



