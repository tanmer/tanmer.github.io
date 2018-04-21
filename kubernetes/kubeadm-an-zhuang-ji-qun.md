# Kubeadm安装集群

官方文档：[https://kubernetes.io/docs/setup/independent/install-kubeadm/](https://kubernetes.io/docs/setup/independent/install-kubeadm/)

## 安装Docker 1.12

```bash
sudo -s
curl https://releases.rancher.com/install-docker/1.12.sh | sh
```

脚本来自Rancher [https://rancher.com/docs/rancher/v1.6/en/hosts/\#supported-docker-versions](https://rancher.com/docs/rancher/v1.6/en/hosts/#supported-docker-versions)

## 安装kubeadm和kubectl

这里需要翻墙，参考 [Ubuntu](https://doc.tanmer.cn/ubuntu) 中的翻墙技巧

```bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

检查`Docker`是否使用`Cgroup Driver`

```bash
docker info | grep -i cgroup
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

为了让docker和k8s的Cgroup Driver保持一致，在参数后面加上 `--cgroup-driver=cgroupfs`

## 创建Master节点

创建之前，先关闭swap

```bash
sudo swapoff -a
# 注释掉swap分区
sudo vi /etc/fstab
```

### 预初始化集群

{% hint style="info" %}
这里说预初始化集群，是因为这次不是真正的初始化，只是借助他找出依赖的Docker镜像
{% endhint %}

因为k8s的镜像都在Google服务器行，我们不翻墙是无法访问的。

```bash
sudo kubeadm init
```

### 监视Docker日志

执行之后，新开窗口，监控docker日志，记录哪些镜像下载失败，并记录下来

```bash
journalctl -u docker -f
```

输出大概是这样的

```text
Apr 21 21:40:25 node1 dockerd[1144]: time="2018-04-21T21:40:25.098887386+08:00" level=error msg="Handler for GET /v1.24/images/k8s.gcr.io/pause-amd64:3.1/json returned error: No such image: k8s.gcr.io/pause-amd64:3.1"
Apr 21 21:40:28 node1 dockerd[1144]: time="2018-04-21T21:40:28.515676456+08:00" level=error msg="Handler for GET /v1.24/images/k8s.gcr.io/pause-amd64:3.1/json returned error: No such image: k8s.gcr.io/pause-amd64:3.1"
Apr 21 21:40:28 node1 dockerd[1144]: time="2018-04-21T21:40:28.528631577+08:00" level=error msg="Handler for GET /v1.24/images/k8s.gcr.io/pause-amd64:3.1/json returned error: No such image: k8s.gcr.io/pause-amd64:3.1"
Apr 21 21:40:28 node1 dockerd[1144]: time="2018-04-21T21:40:28.535282319+08:00" level=error msg="Handler for GET /v1.24/images/k8s.gcr.io/pause-amd64:3.1/json returned error: No such image: k8s.gcr.io/pause-amd64:3.1"
Apr 21 21:40:28 node1 dockerd[1144]: time="2018-04-21T21:40:28.544880690+08:00" level=error msg="Handler for GET /v1.24/images/k8s.gcr.io/pause-amd64:3.1/json returned error: No such image: k8s.gcr.io/pause-amd64:3.1"
```

{% hint style="info" %}
这个窗口不要关，后面还有好几个镜像需要手动从国外下载。
{% endhint %}

回到刚才的`kubeadm init`窗口，按⌃c终止任务，然后通过如下命令找出需要下载的Docker镜像

```text
root@node1:/etc# grep image /etc/kubernetes/manifests/*
/etc/kubernetes/manifests/etcd.yaml:    image: k8s.gcr.io/etcd-amd64:3.1.12
/etc/kubernetes/manifests/kube-apiserver.yaml:    image: k8s.gcr.io/kube-apiserver-amd64:v1.10.1
/etc/kubernetes/manifests/kube-controller-manager.yaml:    image: k8s.gcr.io/kube-controller-manager-amd64:v1.10.1
/etc/kubernetes/manifests/kube-scheduler.yaml:    image: k8s.gcr.io/kube-scheduler-amd64:v1.10.1
```

目前，我们就找出了如下5个用到的镜像：

```text
k8s.gcr.io/pause-amd64:3.1
k8s.gcr.io/etcd-amd64:3.1.12
k8s.gcr.io/kube-apiserver-amd64:v1.10.1
k8s.gcr.io/kube-controller-manager-amd64:v1.10.1
k8s.gcr.io/kube-scheduler-amd64:v1.10.1
```

现在，找一台国外的服务器，加载上面的镜像，推送到国内自己的Gitlab Docker Registry

```bash
# 从国外下载镜像

images=(
k8s.gcr.io/pause-amd64:3.1
k8s.gcr.io/etcd-amd64:3.1.12
k8s.gcr.io/kube-apiserver-amd64:v1.10.1
k8s.gcr.io/kube-controller-manager-amd64:v1.10.1
k8s.gcr.io/kube-scheduler-amd64:v1.10.1
k8s.gcr.io/kube-proxy-amd64:v1.10.1
)

for sourceImageName in ${images[@]} ; do
    targetImageName=docker.corp.tanmer.com/tanmer/dockers/${sourceImageName}
    (    docker pull ${sourceImageName} \
      && docker tag ${sourceImageName} ${targetImageName} \
      && docker push ${targetImageName} \
      && docker rmi ${targetImageName} \
    ) || break
done
```

现在，我们回到刚才要安装`kubeadm init`的服务器，执行下面命令，下载镜像

```bash
# 恢复镜像到国内

images=(
k8s.gcr.io/pause-amd64:3.1
k8s.gcr.io/etcd-amd64:3.1.12
k8s.gcr.io/kube-apiserver-amd64:v1.10.1
k8s.gcr.io/kube-controller-manager-amd64:v1.10.1
k8s.gcr.io/kube-scheduler-amd64:v1.10.1
k8s.gcr.io/kube-proxy-amd64:v1.10.1
)

for targetImageName in ${images[@]} ; do
    sourceImageName=docker.corp.tanmer.com/tanmer/dockers/${targetImageName}
    (    docker pull ${sourceImageName} \
      && docker tag ${sourceImageName} ${targetImageName} \
      && docker rmi ${sourceImageName} \
    ) || break
done
```

### 正式初始化集群

```bash
# 重置刚才的集群命令
kubeadm reset
# 重新初始化
kubeadm init --pod-network-cidr=10.244.0.0/16
```

{% hint style="info" %}
这里用到了参数`--pod-network-cidr=10.244.0.0/16` 是因为我们使用`Flannel`网络，会在后面配置。
{% endhint %}

初始化成功之后，输入如下：

```text
root@node1:~# kubeadm init --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.10.1
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
	[WARNING FileExisting-crictl]: crictl not found in system path
Suggestion: go get github.com/kubernetes-incubator/cri-tools/cmd/crictl
[preflight] Starting the kubelet service
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [node1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.1.10]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated etcd/ca certificate and key.
[certificates] Generated etcd/server certificate and key.
[certificates] etcd/server serving cert is signed for DNS names [localhost] and IPs [127.0.0.1]
[certificates] Generated etcd/peer certificate and key.
[certificates] etcd/peer serving cert is signed for DNS names [node1] and IPs [10.0.1.10]
[certificates] Generated etcd/healthcheck-client certificate and key.
[certificates] Generated apiserver-etcd-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
[init] This might take a minute or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 21.002070 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node node1 as master by adding a label and a taint
[markmaster] Master node1 tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: sxs6qp.neypsgvrkx4whmqm
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 10.0.1.10:6443 --token sxs6qp.neypsgvrkx4whmqm --discovery-token-ca-cert-hash sha256:242ca9d395b945ca774b28dbfc617f250152ae0a9b700c55be682f6f35a53edd

```

根据输出提示，我们拷贝kube配置文件，测试通过`kubectl`获取集群状态。

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

xiaohui@node1:~$ kubectl cluster-info
Kubernetes master is running at https://10.0.1.10:6443
KubeDNS is running at https://10.0.1.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## 加入worker节点

### 安装Docker、kubeadm和kubectl

登录`node2`服务器，按照上面步骤安装`Docker`、`kubeadm`和`kubectl`

执行Master节点的输出提示：

```bash
kubeadm join 10.0.1.10:6443 --token sxs6qp.neypsgvrkx4whmqm --discovery-token-ca-cert-hash sha256:242ca9d395b945ca774b28dbfc617f250152ae0a9b700c55be682f6f35a53edd
```

这里也可以用journalctl -u docker -f查看日志，知道哪些镜像下载失败。

### 下载镜像

```bash
images=(
k8s.gcr.io/pause-amd64:3.1
k8s.gcr.io/kube-proxy-amd64:v1.10.1
)

for targetImageName in ${images[@]} ; do
    sourceImageName=docker.corp.tanmer.com/tanmer/dockers/${targetImageName}
    (    docker pull ${sourceImageName} \
      && docker tag ${sourceImageName} ${targetImageName} \
      && docker rmi ${sourceImageName} \
    ) || break
done
```

## 安装Flannel网络

现在，我们回到Master节点，查看节点状态，会发现他们的状态都是NotReady，这是因为我们还没有安装网络插件。

{% code-tabs %}
{% code-tabs-item title="On node1" %}
```bash
xiaohui@node1:~$ kubectl get no
NAME      STATUS     ROLES     AGE       VERSION
node1     NotReady   master    23m       v1.10.1
node2     NotReady   <none>    5m        v1.10.1
```
{% endcode-tabs-item %}
{% endcode-tabs %}

继续翻墙下载镜像

```bash
# 从国外下载镜像

images=(
k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.8
k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.8
k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.8
)

for sourceImageName in ${images[@]} ; do
    targetImageName=docker.corp.tanmer.com/tanmer/dockers/${sourceImageName}
    (    docker pull ${sourceImageName} \
      && docker tag ${sourceImageName} ${targetImageName} \
      && docker push ${targetImageName} \
      && docker rmi ${targetImageName} \
    ) || break
done

# 恢复镜像到国内

images=(
k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.8
k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.8
k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.8
)

for targetImageName in ${images[@]} ; do
    sourceImageName=docker.corp.tanmer.com/tanmer/dockers/${targetImageName}
    (    docker pull ${sourceImageName} \
      && docker tag ${sourceImageName} ${targetImageName} \
      && docker rmi ${sourceImageName} \
    ) || break
done

# 从国外下载镜像

docker pull quay.io/coreos/flannel:v0.9.1-amd64
docker tag quay.io/coreos/flannel:v0.9.1-amd64 docker.corp.tanmer.com/tanmer/dockers/flannel:v0.9.1-amd64
docker push docker.corp.tanmer.com/tanmer/dockers/flannel:v0.9.1-amd64
docker rmi docker.corp.tanmer.com/tanmer/dockers/flannel:v0.9.1-amd64

# 恢复镜像到国内

docker pull docker.corp.tanmer.com/tanmer/dockers/flannel:v0.9.1-amd64
docker tag docker.corp.tanmer.com/tanmer/dockers/flannel:v0.9.1-amd64 quay.io/coreos/flannel:v0.9.1-amd64
docker rmi docker.corp.tanmer.com/tanmer/dockers/flannel:v0.9.1-amd64

```

现在查看集群状态，所有`STATUS`都变成了`Ready`

```bash
xiaohui@node1:~$ kubectl get no
NAME      STATUS    ROLES     AGE       VERSION
node1     Ready     master    51m       v1.10.1
node2     Ready     <none>    34m       v1.10.1
```



