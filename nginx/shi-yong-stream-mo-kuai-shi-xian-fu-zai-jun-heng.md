# 使用stream模块实现负载均衡

修改nginx配置文件，从`stream-enabled`目录中读取负载均衡的配置文件

```bash
sudo vi /etc/nginx/nginx.conf
```

{% code-tabs %}
{% code-tabs-item title="/etc/nginx/nginx.conf" %}
```text
stream {
        include /etc/nginx/stream-enabled/*;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
sudo mkdir /etc/nginx/stream-enabled/
sudo mkdir /etc/nginx/stream-available/
```

创建一个负载均衡配置文件：

```bash
sudo vi /etc/nginx/stream-available/kubernetes
```

```text
upstream kubernetes {
	server 10.100.0.117:6443;
	server 10.100.0.118:6443;
}

server {
	listen 6443;
	proxy_pass kubernetes;
}
```

检查端口`6443`是否开始监听

```text
$ netstat -nptl

roto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      891/sshd
tcp        0      0 0.0.0.0:6443            0.0.0.0:*               LISTEN      27342/nginx -g daem
```

检查数据是否转发过来：

```text
$ curl -k https://localhost:6443
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {

  },
  "code": 403
}
```



