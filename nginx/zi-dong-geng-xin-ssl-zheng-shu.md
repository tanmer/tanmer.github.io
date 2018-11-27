# 自动更新SSL证书

## 第一步，安装certbot

```bash
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
```

## 第二步，生成证书

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

{% hint style="warning" %}
注意：此命令会自动搜索出对应的nginx虚拟主机配置文件，然后替换SSL相关的证书地址，执行命令之前，建议先备份配置文件（如：/etc/nginx/site-enabled/example.com）

端口必须是`80` Let's encrpyt不支持80/443端口以外的验证。如果是其他端口，唯一办法就是修改用DNS TXT解析的方法实现。这个不在Certbot工具的功能范围呢。
{% endhint %}

如果是第一次运行 `certbot`，会提示我们输入email并同意使用协议，这里email建议使用真实地址，方便接收证书过期提示。

## 第三步，定期更新证书

用apt-get install certbot之后，系统会安装一个服务和一个定时器，每天certbot会启动两次，会提前1个月为域名续期。

```bash
root@10-10-21-131:~# systemctl status certbot.service
● certbot.service - Certbot
   Loaded: loaded (/lib/systemd/system/certbot.service; static; vendor preset: enabled)
   Active: inactive (dead)
     Docs: file:///usr/share/doc/python-certbot-doc/html/index.html
           https://letsencrypt.readthedocs.io/en/latest/
root@10-10-21-131:~# systemctl status certbot.timer
● certbot.timer - Run certbot twice daily
   Loaded: loaded (/lib/systemd/system/certbot.timer; enabled; vendor preset: enabled)
   Active: active (waiting) since Wed 2018-10-17 10:57:51 CST; 31min ago
```

## 结论

在Ubuntu上安装的certbot服务，会自动更新证书，一次配置，以后再也不用担心证书过期问题了。操作非常简单，5分钟之内即可完成上面操作。

参考：

[https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)

