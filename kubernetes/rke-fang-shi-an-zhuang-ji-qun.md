# rke方式安装Kubernetes

上面我们知道了有两种方式安装集群：`kubeadm`和`rancher`，第一种防止如果是在国外服务器，是最简单的部署方式，但是到了国内，因为`gcr.io`被墙，就对部署造成了很大麻烦。第二种方式，Rancher 1.x目前只可以部署1.8.5版本的集群，无法体验更成熟的1.10版本，还有几个问题也是让我不喜欢用Rancher的原因：

1. 必须安装Rancher Server，浪费了一台服务器
2. Rancher的网络组件性能问题，应该没有Flannel快
3. 权限控制方面，要用kubectl控制集群，必须分配一个Rancher可编辑权限的帐号，这个帐号可以在Rancher的Web端进入任何一个容器，让Kubernetes的 RBAC 大打折扣。

因为Rancher 1.x的这些原因，Rancher推出了一个部署k8s原生集群的工具`rke`，他可以解决墙的问题，自动帮我们配置`etcd`集群，master集群，实现高可用。自动帮我们增加、删除节点，让集群管理变得非常简单。

下面我们开始正题

## 准备服务器节点

至少3台服务器，保证etcd数据高可用

每台服务器配置ssh密钥登录，让当前运行rke程序的电脑能够无密码登录3台服务器

```text
mkdir ~/.ssh
vi ~/.ssh/authorized_keys
```

把本地公钥`~/.ssh/id_rsa.pub`内容粘贴进3台服务器上的`~/.ssh/authorized_keys`文件

## 下载rke

rke 的github地址，[https://github.com/rancher/rke](https://github.com/rancher/rke) 在这里，我们可以下载最新版本的rke二进制文件

## 配置集群

下面命令，根据一路提示，即可完成配置，创建`cluster.yml`文件，这个文件一定要保存好，因为以后升级集群就靠他了。

```bash
rke config
```

这里有一点比较迷糊的得放得注意，`[+] Cluster Level SSH Private Key Path [~/.ssh/id_rsa]:`这个是说运行`rke`的电脑登录k8s节点的密钥文件地址，后面每个Host都会提示的`[+] SSH Private Key Path of host (192.180.10.1) [none]:`表示如果这台节点的登录密钥不是上面填的`Cluster Level`密钥，那就在这里指定另外一个文件，否者就按回车默认。

生成cluster.yml文件之后，手工添加以下配置：

```yaml
 kubelet:
    image: rancher/hyperkube:v1.10.1-rancher1
    extra_args:
      read-only-port: 10255
```

这个`read-only-port`可以让后面安装的`Heapster`能够获取到节点的硬件资源占用情况。



