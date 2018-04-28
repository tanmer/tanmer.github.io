# RBAC用户管理

## 方式认证

### X509 Client Certs

#### 创建用户

参考文档： [https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/](https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/)

**第一步：生成用户证书**

在k8s master节点执行：

```bash
sudo -s
mkdir -p ~/k8s-user-certs
cd ~/k8s-user-certs
cert_user=wenlg
cert_group=deployer
openssl genrsa -out ${cert_user}.key 2048
openssl req -new -key ${cert_user}.key -out ${cert_user}.csr -subj "/CN=${cert_user}/O=${cert_group}"
openssl x509 -req -in ${cert_user}.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ${cert_user}.crt -days 365
```

下载生成的证书`wenlg.crt`和密钥`wenlg.key`，保存在自己电脑上，推荐存放在`~/.certs/`目录

在本地电脑执行：

```bash
cert_user=wenlg
kubectl config set-credentials ${cert_user} --client-certificate=$(realpath ~/.certs/${cert_user}.crt)  --client-key=$(realpath ~/.certs/${cert_user}.key)
kubectl config set-context ${cert_user}@kubernetes --cluster=kubernetes --user=${cert_user}
```

本地执行，如果提示`Error from server (Forbidden): pods is forbidden: User "wenlg" cannot list pods in the namespace "default" `就说明证书可以了。

```bash
kubectl --context=${cert_user}@kubernetes get pods
```

**配置权限**

创建一个角色`deployment-manager`：

```bash
cat <<EOS|kubectl apply -f -
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: deployment-manager
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
EOS
```

绑定角色和用户：

```bash
cat <<EOS|kubectl apply -f -
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: deployment-manager-binding
  namespace: default
subjects:
- kind: User
  name: ${cert_user}
  apiGroup: ""
roleRef:
  kind: Role
  name: deployment-manager
  apiGroup: ""
EOS
```

 测试一下权限：

```bash
# 部署一个镜像
kubectl --context=${cert_user}@kubernetes run --image citizenstig/httpbin httpbin
# 显示部署和pod列表
kubectl --context=${cert_user}@kubernetes get deploy,po
```

{% code-tabs %}
{% code-tabs-item title="Output" %}
```text
➜  ~ kubectl --context=${cert_user}@kubernetes get deploy,po
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/httpbin   1         1         1            1           47s

NAME                          READY     STATUS    RESTARTS   AGE
po/httpbin-5c5449d8b8-qr2dz   1/1       Running   0          47s
```
{% endcode-tabs-item %}
{% endcode-tabs %}



