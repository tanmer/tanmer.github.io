# Traefik配置

官方文档：[https://github.com/containous/traefik/blob/master/docs/configuration/backends/kubernetes.md](https://github.com/containous/traefik/blob/master/docs/configuration/backends/kubernetes.md)

## 安装Consul

因为我们要配置Traefik高可用，数据存储就得用到Consul KV Store。

给3台服务器打上标签，`Consul`会以`StatefulSet`模式安装在上面

```bash
kubectl label nodes node1 node2 node2 consul=server
```

创建服务

```bash
cat <<YAML | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: consul
  namespace: kube-system
  labels:
    app: consul
spec:
  clusterIP: None
  ports:
    - name: http
      port: 8500
      targetPort: 8500
    - name: https
      port: 8443
      targetPort: 8443
    - name: rpc
      port: 8400
      targetPort: 8400
    - name: serflan-tcp
      protocol: "TCP"
      port: 8301
      targetPort: 8301
    - name: serflan-udp
      protocol: "UDP"
      port: 8301
      targetPort: 8301
    - name: serfwan-tcp
      protocol: "TCP"
      port: 8302
      targetPort: 8302
    - name: serfwan-udp
      protocol: "UDP"
      port: 8302
      targetPort: 8302
    - name: server
      port: 8300
      targetPort: 8300
    - name: consuldns
      port: 8600
      targetPort: 8600
  selector:
    app: consul
YAML
```

创建`StatefulSet`，数据存储在节点的`/mnt/consul`目录

```bash
cat <<YAML | kubectl apply -f -
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: consul
  namespace: kube-system
spec:
  serviceName: consul
  replicas: 3
  template:
    metadata:
      labels:
        app: consul
    spec:
      nodeSelector:
        consul: server
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: consul
                    operator: In
                    values:
                      - server
              topologyKey: kubernetes.io/hostname
      terminationGracePeriodSeconds: 10
      securityContext:
        fsGroup: 1000
      containers:
      - name: consul
        image: consul:0.9.2
        args:
        - "agent"
        - "-advertise=\$(POD_IP)"
        - "-bind=0.0.0.0"
        - "-bootstrap-expect=3"
        - "-retry-join=consul-0.consul.\$(NAMESPACE).svc.cluster.local"
        - "-retry-join=consul-1.consul.\$(NAMESPACE).svc.cluster.local"
        - "-retry-join=consul-2.consul.\$(NAMESPACE).svc.cluster.local"
        - "-client=0.0.0.0"
        - "-datacenter=fr"
        - "-data-dir=/consul/data"
        - "-domain=cluster.local"
        - "-server"
        - "-ui"
        - "-disable-host-node-id"
        ports:
        - containerPort: 8500
          name: ui-port
        - containerPort: 8400
          name: alt-port
        - containerPort: 53
          name: udp-port
        - containerPort: 8443
          name: https-port
        - containerPort: 8080
          name: http-port
        - containerPort: 8301
          name: serflan
        - containerPort: 8302
          name: serfwan
        - containerPort: 8600
          name: consuldns
        - containerPort: 8300
          name: server
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        lifecycle:
          preStop:
            exec:
              command:
              - /bin/sh
              - -c
              - consul leave
        volumeMounts:
        - name: ca-certificates
          mountPath: /etc/ssl/certs
        - name: consul-data
          mountPath: /consul/data
      volumes:
      - name: ca-certificates
        hostPath:
          path: /usr/share/ca-certificates/
      - name: consul-data
        hostPath:
          path: /mnt/consul
YAML
```

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



