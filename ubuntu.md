# Ubuntu

## 修改apt源

```bash
sudo sed -i 's!http://us.archive.ubuntu.com!https://mirrors.tuna.tsinghua.edu.cn!' /etc/apt/sources.list
```

## 翻墙技巧

### 傻瓜式

[https://github.com/googlehosts/hosts](https://github.com/googlehosts/hosts)

下载hosts文件，内容添加到`/etc/hosts`

### 进阶式

下载上面仓库的[dnsmasq.conf](https://github.com/googlehosts/hosts/raw/master/hosts-files/dnsmasq.conf)文件，安装dnsmasq服务

```bash
sudo apt-get install dnsmasq
sudo mv ~/dnsmasq.conf /etc/dnsmasq.conf
```

## 安装软件

### ping

```bash
apt install iputils-ping -y
```

### nslookup

```bash
apt install dnsutils -y
```



