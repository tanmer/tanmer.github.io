# Kubeapps

`Kubeapps` 是集`CLI`和`GUI`一体的`Helm` charts部署工具。比如一个复杂的项目，有API端，后端，前端，Worker端，每个端都是需要独立部署，他们都有各自的chart，要一次性部署好这几个端，通过`kubeapps`一个命令就可以搞定。

## 下载

官方文档：[https://github.com/kubeapps/kubeapps/blob/master/docs/getting-started.md\#installation-of-the-kubeapps-installer](https://github.com/kubeapps/kubeapps/blob/master/docs/getting-started.md#installation-of-the-kubeapps-installer)

在`github`的[`release`](https://github.com/kubeapps/kubeapps/releases)页面，下载最新版，放到bin目录里

```bash
mv ~/Downloads/kubeapps-darwin-amd64 /usr/local/bin/kubeapps
chmod +x /usr/local/bin/kubeapps
```

## 安装

```bash
kubeapps up
```

{% code-tabs %}
{% code-tabs-item title="Output" %}
```text
➜  tamigos kubeapps up
INFO[0000] Fetching schemas for 44 resources
INFO[0002] Updating customresourcedefinitions apprepositories.kubeapps.com
INFO[0002]  Creating non-existent customresourcedefinitions apprepositories.kubeapps.com
INFO[0003] Updating clusterroles apprepository-controller
INFO[0003]  Creating non-existent clusterroles apprepository-controller
INFO[0003] Updating clusterrolebindings apprepository-controller
INFO[0003]  Creating non-existent clusterrolebindings apprepository-controller
INFO[0003] Updating customresourcedefinitions cronjobtriggers.kubeless.io
INFO[0003]  Creating non-existent customresourcedefinitions cronjobtriggers.kubeless.io
INFO[0003] Updating customresourcedefinitions functions.kubeless.io
INFO[0003]  Creating non-existent customresourcedefinitions functions.kubeless.io
INFO[0003] Updating customresourcedefinitions httptriggers.kubeless.io
INFO[0003]  Creating non-existent customresourcedefinitions httptriggers.kubeless.io
INFO[0003] Updating namespaces kubeapps
INFO[0003]  Creating non-existent namespaces kubeapps
INFO[0003] Updating namespaces kubeless
INFO[0003]  Creating non-existent namespaces kubeless
INFO[0003] Updating clusterroles kubeless-controller-deployer
INFO[0003]  Creating non-existent clusterroles kubeless-controller-deployer
INFO[0003] Updating clusterrolebindings kubeless-controller-deployer
INFO[0004]  Creating non-existent clusterrolebindings kubeless-controller-deployer
INFO[0004] Updating clusterrolebindings sealed-secrets-controller
INFO[0004]  Creating non-existent clusterrolebindings sealed-secrets-controller
INFO[0004] Updating customresourcedefinitions sealedsecrets.bitnami.com
INFO[0004]  Creating non-existent customresourcedefinitions sealedsecrets.bitnami.com
INFO[0004] Updating clusterroles secrets-unsealer
INFO[0004]  Creating non-existent clusterroles secrets-unsealer
INFO[0004] Updating clusterrolebindings tiller-cluster-admin
INFO[0004]  Creating non-existent clusterrolebindings tiller-cluster-admin
INFO[0004] Updating customresourcedefinitions kubeapps.helmreleases.helm.bitnami.com
INFO[0004]  Creating non-existent customresourcedefinitions kubeapps.helmreleases.helm.bitnami.com
INFO[0004] Updating rolebindings kube-system.sealed-secrets-controller
INFO[0004]  Creating non-existent rolebindings kube-system.sealed-secrets-controller
INFO[0004] Updating services kube-system.sealed-secrets-controller
INFO[0004]  Creating non-existent services kube-system.sealed-secrets-controller
INFO[0004] Updating serviceaccounts kube-system.sealed-secrets-controller
INFO[0005]  Creating non-existent serviceaccounts kube-system.sealed-secrets-controller
INFO[0005] Updating roles kube-system.sealed-secrets-key-admin
INFO[0005]  Creating non-existent roles kube-system.sealed-secrets-key-admin
INFO[0005] Updating roles kubeapps.apprepository-controller
INFO[0005]  Creating non-existent roles kubeapps.apprepository-controller
INFO[0005] Updating rolebindings kubeapps.apprepository-controller
INFO[0005]  Creating non-existent rolebindings kubeapps.apprepository-controller
INFO[0005] Updating serviceaccounts kubeapps.apprepository-controller
INFO[0005]  Creating non-existent serviceaccounts kubeapps.apprepository-controller
INFO[0005] Updating services kubeapps.chartsvc
INFO[0005]  Creating non-existent services kubeapps.chartsvc
INFO[0005] Updating apprepositories kubeapps.incubator
INFO[0005]  Creating non-existent apprepositories kubeapps.incubator
INFO[0005] Updating services kubeapps.kubeapps
INFO[0006]  Creating non-existent services kubeapps.kubeapps
INFO[0006] Updating services kubeapps.kubeapps-dashboard-ui
INFO[0006]  Creating non-existent services kubeapps.kubeapps-dashboard-ui
INFO[0006] Updating configmaps kubeapps.kubeapps-dashboard-ui-vhost-425de41
INFO[0006]  Creating non-existent configmaps kubeapps.kubeapps-dashboard-ui-vhost-425de41
INFO[0006] Updating configmaps kubeapps.kubeapps-vhost-5811f90
INFO[0006]  Creating non-existent configmaps kubeapps.kubeapps-vhost-5811f90
INFO[0006] Updating secrets kubeapps.mongodb
INFO[0006]  Creating non-existent secrets kubeapps.mongodb
INFO[0006] Updating services kubeapps.mongodb
INFO[0006]  Creating non-existent services kubeapps.mongodb
INFO[0006] Updating persistentvolumeclaims kubeapps.mongodb-data
INFO[0006]  Creating non-existent persistentvolumeclaims kubeapps.mongodb-data
INFO[0007] Updating apprepositories kubeapps.stable
INFO[0007]  Creating non-existent apprepositories kubeapps.stable
INFO[0007] Updating apprepositories kubeapps.svc-cat
INFO[0007]  Creating non-existent apprepositories kubeapps.svc-cat
INFO[0007] Updating serviceaccounts kubeapps.tiller
INFO[0007]  Creating non-existent serviceaccounts kubeapps.tiller
INFO[0007] Updating serviceaccounts kubeless.controller-acct
INFO[0007]  Creating non-existent serviceaccounts kubeless.controller-acct
INFO[0007] Updating configmaps kubeless.kubeless-config
INFO[0008]  Creating non-existent configmaps kubeless.kubeless-config
INFO[0008] Updating deployments kube-system.sealed-secrets-controller
INFO[0008]  Creating non-existent deployments kube-system.sealed-secrets-controller
INFO[0008] Updating deployments kubeapps.apprepository-controller
INFO[0008]  Creating non-existent deployments kubeapps.apprepository-controller
INFO[0008] Updating deployments kubeapps.chartsvc
INFO[0008]  Creating non-existent deployments kubeapps.chartsvc
INFO[0008] Updating deployments kubeapps.kubeapps
INFO[0008]  Creating non-existent deployments kubeapps.kubeapps
INFO[0008] Updating deployments kubeapps.kubeapps-dashboard-ui
INFO[0008]  Creating non-existent deployments kubeapps.kubeapps-dashboard-ui
INFO[0008] Updating deployments kubeapps.mongodb
INFO[0009]  Creating non-existent deployments kubeapps.mongodb
INFO[0009] Updating deployments kubeapps.tiller-deploy
INFO[0009]  Creating non-existent deployments kubeapps.tiller-deploy
INFO[0009] Updating deployments kubeless.kubeless-controller-manager
INFO[0009]  Creating non-existent deployments kubeless.kubeless-controller-manager

Kubeapps has been deployed successfully.
It may take a few minutes for all components to be ready.

NAMESPACE  	NAME                         	CLUSTER-IP   	EXTERNAL-IP	PORT(S)
kubeapps   	svc/chartsvc                 	10.43.208.41 	           	8080/TCP,
kubeapps   	svc/kubeapps                 	10.43.197.11 	           	8080/TCP,
kubeapps   	svc/kubeapps-dashboard-ui    	10.43.229.120	           	8080/TCP,
kubeapps   	svc/mongodb                  	10.43.136.139	           	27017/TCP,
kube-system	svc/sealed-secrets-controller	10.43.208.151	           	8080/TCP,

NAMESPACE  	NAME                              	DESIRED	CURRENT	UP-TO-DATE	AVAILABLE
kubeapps   	deploy/apprepository-controller   	1      	1      	1         	0
kubeapps   	deploy/chartsvc                   	1      	1      	1         	0
kubeapps   	deploy/kubeapps                   	1      	1      	1         	0
kubeapps   	deploy/kubeapps-dashboard-ui      	1      	1      	1         	0
kubeapps   	deploy/mongodb                    	1      	1      	1         	0
kubeapps   	deploy/tiller-deploy              	1      	1      	1         	0
kubeless   	deploy/kubeless-controller-manager	1      	1      	1         	0
kube-system	deploy/sealed-secrets-controller  	1      	1      	1         	0

NAMESPACE	NAME	DESIRED	CURRENT

NAMESPACE	NAME                                         	STATUS
kubeapps 	pod/apprepository-controller-7f4dc847df-jflhb	Pending
kubeapps 	pod/chartsvc-64f494f5f7-dzs49                	Pending
kubeapps 	pod/kubeapps-594c9d4fb9-w6skb                	Pending
kubeapps 	pod/kubeapps-dashboard-ui-9bb44d58b-tf2bn    	Pending
kubeapps 	pod/mongodb-55b55565ff-9tvj5                 	Pending

You can run `kubectl get all --all-namespaces -l created-by=kubeapps` to check the status of the Kubeapps components.
```
{% endcode-tabs-item %}
{% endcode-tabs %}

庆幸的是，kubeapps用的docker镜像不需要翻墙，有点不适应。

检查





