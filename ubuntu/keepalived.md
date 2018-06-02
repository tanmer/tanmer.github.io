# Keepalived

Keepalived 结合VIP，可以帮助我们实现IP自动漂移，当一台主机下线，IP会漂移到另外一台主机上，实现IP上的服务高可用

## 安装

```bash
apt update
apt install keepalived -y
```

## 配置

下面配置的`10.9.181.169`是VIP地址

{% code-tabs %}
{% code-tabs-item title="/etc/keepalived/keepalived.conf" %}
```text
global_defs {
  vrrp_version 3
  vrrp_iptables KEEPALIVED-VIP
}

vrrp_instance vips {
  state BACKUP
  interface eth0
  virtual_router_id 50
  priority 100
  nopreempt
  advert_int 1

  track_interface {
    eth0
  }

  virtual_ipaddress {
    10.9.181.169
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 服务生效

```bash
systemctl restart keepalived
```

{% hint style="info" %}
注意：每台需要IP漂移到的目标主机，都要执行以上安装和配置步骤。
{% endhint %}

## 排错

在云主机上测试，很可能Keepalived不会自动漂移VIP，查看日志发现每天主机都变成了Master

```text
RRP_Instance(vips) Transition to MASTER STATE
VRRP_Instance(vips) Entering MASTER STATE
```

这很可能是因为网络交换机过滤了`multicast`数据，参考 [https://serverfault.com/questions/512153/both-servers-running-keepalived-become-master-and-have-a-same-virtual-ip](https://serverfault.com/questions/512153/both-servers-running-keepalived-become-master-and-have-a-same-virtual-ip)

如果无法关闭多播的过滤，那么可以用`unicast_peer`指定广播到Keepalived主机的IP地址列表

```text
vrrp_instance VI_1 {
  state MASTER
  interface eth0
  #unicast peer 格式必须完全匹配！否则会起不来，必须写成三行。
  unicast_peer {
    192.168.0.10
    192.168.0.11
    192.168.0.12
  }
  ...
```



