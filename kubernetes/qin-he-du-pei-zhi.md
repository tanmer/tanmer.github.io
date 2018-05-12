# 亲和度配置

## Node亲和度

### 限制Pod调度到Node

```bash
kubectl taint node 10.100.0.133 backend=true:NoSchedule
```

这个命令表示在`node` `10.100.0.133`上，只有`pod`打上标签`backend=true`才能被调度到这个节点。

## Pod亲和度

