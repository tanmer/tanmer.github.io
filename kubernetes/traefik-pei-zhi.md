# Traefik配置

官方文档：[https://github.com/containous/traefik/blob/master/docs/configuration/backends/kubernetes.md](https://github.com/containous/traefik/blob/master/docs/configuration/backends/kubernetes.md)

## 部署traefik

执行下面代码，即可以部署`traefix`

```bash
kubectl apply -f traefik.yml
```

配置文件中，需要注意的的是`ConfigMap`中，acme配置段中的`email`地址，这个地址是需要通过[`certbot`](https://certbot.eff.org/lets-encrypt/osx-other)去注册的，不然无法使用自动tls证书更新的功能。

{% code-tabs %}
{% code-tabs-item title="traefik.yml" %}
```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ingress
  namespace: kube-system

---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: ingress
subjects:
  - kind: ServiceAccount
    name: ingress
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io

---
kind: ConfigMap
apiVersion: v1
data:
  traefik.toml: |
    checkNewVersion = false
    IdleTimeout = "180s"
    MaxIdleConnsPerHost = 500
    logLevel = "INFO"
    defaultEntryPoints = ["http", "https"]

    [retry]
    attempts = 3

    [web]
    address = ":8080"

    [entryPoints]
      [entryPoints.http]
      address = ":80"
      #  [entryPoints.http.redirect]
      #  entryPoint = "https"
      [entryPoints.https]
      address = ":443"
        [entryPoints.https.tls]

    [acme]
    email = "xiaohui@tanmer.com"
    storage = "acme.json"
    entryPoint = "https"
    onDemand = true
    OnHostRule = true
    acmeLogging = true
    [acme.httpChallenge]
    entryPoint = "http"

metadata:
  name: traefik-conf
  namespace: kube-system

---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: traefik-ingress-lb
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      terminationGracePeriodSeconds: 60
      hostNetwork: true
      restartPolicy: Always
      serviceAccountName: ingress
      volumes:
      - name: config
        configMap:
          name: traefik-conf
      containers:
      - image: traefik
        name: traefik-ingress-lb
        resources:
          limits:
            cpu: 200m
            memory: 30Mi
          requests:
            cpu: 100m
            memory: 20Mi
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: https
          containerPort: 443
          hostPort: 443
        - name: admin
          containerPort: 8580
          hostPort: 8580
        volumeMounts:
        - mountPath: "/config"
          name: config
        args:
        - --web
        - --web.address=:8580
        - --kubernetes
        - --configfile=/config/traefik.toml
      nodeSelector:
        edgenode: "true"
---
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - name: web
    port: 80
    targetPort: 8580
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
  annotations:
    traefik.ingress.kubernetes.io/frontend-entry-points: http
spec:
  rules:
  - host: traefik.tamigos.in
    http:
      paths:
      - path: /
        backend:
          serviceName: traefik-web-ui
          servicePort: web
```
{% endcode-tabs-item %}
{% endcode-tabs %}

这个配置文件，是不会自动`http`重定向到`https`的，需要在独立的`ingress`中去声明`annotations`

支持的`annotations` 列表：[https://github.com/containous/traefik/blob/master/docs/configuration/backends/kubernetes.md\#general-annotations](https://github.com/containous/traefik/blob/master/docs/configuration/backends/kubernetes.md#general-annotations)

重定向`http`到`https`: `traefik.ingress.kubernetes.io/redirect-entry-point: https`

## 注册Let'sencrypt账号



