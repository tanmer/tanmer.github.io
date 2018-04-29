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



