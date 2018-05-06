# 安装Helm

## 初始化Tiller

`Helm`默认会把`Tiller`安装到`kube-system`空间，我不建议安装在这里，因为后面使用`helm`的用户必须得有`kube-system`空间`pod`的`list`权限，把最核心的pod暴露出来，谁也不愿意。因此，这里建议创建`tiller`空间

```bash
#!/bin/bash

# https://github.com/kubernetes/helm/issues/3460#issuecomment-385992094

set -e
tiller_namespace=tiller # default is kube-system
kubectl describe ns ${tiller_namespace} || kubectl create ns ${tiller_namespace}
kubectl create serviceaccount --namespace ${tiller_namespace} tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=${tiller_namespace}:tiller
helm init --service-account tiller --tiller-image tanmerk8s/tiller:v2.9.0 --upgrade --tiller-namespace ${tiller_namespace}
```

## 配置用户权限

这里`cert_namespace`是需要授权的空间，`tiller_namespace`是我们上面初始化的`Tiller`所在的空间，下面例子给用户`gitlab`添加了对空间`project-staging`的`helm`部署权限。

```bash
#!/bin/bash
set -e

cert_namespace=project-staging
tiller_namespace=tiller

cat <<EOS|kubectl apply -f -
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: tiller-deployers
  namespace: ${tiller_namespace}
rules:
- apiGroups: ["*"]
  resources: ["pods"]
  verbs: ["list"]
- apiGroups: ["*"]
  resources: ["pods/portforward"]
  verbs: ["create"]

---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: deployers
  namespace: ${cert_namespace}
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
EOS

cat <<EOS|kubectl apply -f -
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: deployment
  namespace: ${tiller_namespace}
subjects:
- kind: User
  name: gitlab
  apiGroup: ""
roleRef:
  kind: Role
  name: tiller-deployers
  apiGroup: ""
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: deployment
  namespace: ${cert_namespace}
subjects:
- kind: User
  name: gitlab
  apiGroup: ""
roleRef:
  kind: Role
  name: deployers
  apiGroup: ""
EOS
```

