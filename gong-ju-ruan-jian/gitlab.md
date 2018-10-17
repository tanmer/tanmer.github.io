# Gitlab

## 维护

### 解除IP屏蔽

Gitlab默认开启了[rack-attack](https://github.com/kickstarter/rack-attack/blob/master/README.md)，当Gitlab Runner执行CI，频繁PUSH Docker镜像时，会造成频繁登录，然后IP被Gitlab禁止访问。

解决办法是把Gitlab Runner的IP加入到白名单，然后被禁止的IP从Redis中清除。

{% code-tabs %}
{% code-tabs-item title="/etc/gitlab/gitlab.rb" %}
```ruby
 gitlab_rails['rack_attack_git_basic_auth'] = {
   'enabled' => true,
   'ip_whitelist' => ["127.0.0.1"],
   'maxretry' => 10,
   'findtime' => 60,
   'bantime' => 3600
 }
 
 # gitlab_rails['rack_attack_protected_paths'] = [
#   '/users/password',
#   '/users/sign_in',
#   '/api/#{API::API.version}/session.json',
#   '/api/#{API::API.version}/session',
#   '/users',
#   '/users/confirmation',
#   '/unsubscribes/',
#   '/import/github/personal_access_token'
# ]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

查看屏蔽日志：

```bash
grep "Rack_Attack" /var/log/gitlab/gitlab-rails/production.log

    Rack_Attack: blacklist 182.149.159.201 POST /api/v4/runners
```

进入Redis CLI：

```bash
/opt/gitlab/embedded/bin/redis-cli -h 10.9.61.112
```

查看被禁止的IP：

```text
keys *rack::attack*
```

解除禁止：

```text
del cache:gitlab:rack::attack:allow2ban:ban:182.149.159.201
```

### 每周清理Registry数据

当Gitlab集成CI/CD和Docker Registry之后，每次部署时编译Docker镜像都会产生很多文件，时间长了，一些老的镜像过时了，我们会在Gitlab的Registry页面删除他们，可是这个删除动作不会删除服务器上Docker Registry所在目录的数据。`gitlab-ctl`提供了命令行清理Registry的垃圾数据：

{% code-tabs %}
{% code-tabs-item title="/etc/crontab" %}
```text
0  0    * * 1   root    gitlab-ctl registry-garbage-collect
```
{% endcode-tabs-item %}
{% endcode-tabs %}



