# RBAC集成

官方文档 [https://rancher.com/docs/rancher/v1.6/en/kubernetes/rbac/](https://rancher.com/docs/rancher/v1.6/en/kubernetes/rbac/)

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: edit-dev
  namespace: dev # Specify which namespace you want these permissions granted in
subjects:
  - kind: User
    name: developer1
  - kind: User
    name: developer2
roleRef:
  kind: ClusterRole
  name: edit # Specify which type of role you want the users to have
  apiGroup: rbac.authorization.k8s.io
```

给Namespace `dev`添加编辑人员

