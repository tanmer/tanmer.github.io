# 安装DNSMasq

## 安装

```bash
sudo apt update
sudo apt install dnsmasq
```

## 配置

默认配置，外部电脑是访问DNS `53` 端口的，需要配置`listen-address`

查看本机IP地址：

```text
ubuntu@10-9-124-19:~$ ip addr
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1454 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:32:c3 brd ff:ff:ff:ff:ff:ff
    inet 10.9.124.19/16 brd 10.9.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

然后修改`/etc/dnsmasq.conf`

{% code-tabs %}
{% code-tabs-item title="/etc/dnsmasq.conf" %}
```text
listen-address=127.0.0.1,10.9.124.19
```
{% endcode-tabs-item %}
{% endcode-tabs %}

重启服务

```text
sudo systemctl restart dnsmasq
```

## 测试DNS解析

在网内另一台机器执行nslookup检查DNS解析是否可用

```text
➜  ~ nslookup www.qq.com 10.9.124.19
Server:		10.9.124.19
Address:	10.9.124.19#53

Non-authoritative answer:
Name:	www.qq.com
Address: 180.163.26.39
```

## 简单解决墙内DNS污染

[https://doc.tanmer.cn/ubuntu\#jin-jie-shi](https://doc.tanmer.cn/ubuntu#jin-jie-shi)



