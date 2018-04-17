# 数据库类

## NullDB Connection Adapter

当我们在本地执行`RAILS_ENV=production rails assets:precompile`编译资源时，会提示我们production数据库不存在，通过Gem `activerecord-nulldb-adapter` 可以避免创建数据库

{% code-tabs %}
{% code-tabs-item title="Gemfile" %}
```ruby
gem 'activerecord-nulldb-adapter'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
RAILS_ENV=production DATABASE_ADAPTER=nulldb rails assets:precompile
```

{% code-tabs %}
{% code-tabs-item title="config/database.yml" %}
```yaml
default: &default
  adapter: <%= ENV.fetch('DATABASE_ADAPTER', 'postgresql') %>
  encoding: <%= ENV.fetch('DATABASE_ENCODING', 'unicode') %>
  # For details on connection pooling, see rails configuration guide
  # http://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: <%= ENV['DATABASE_HOST'] %>
  username: <%= ENV['DATABASE_USERNAME'] %>
  password: <%= ENV['DATABASE_PASSWORD'] %>
  port: <%= ENV['DATABASE_PORT'] %>
  timeout: 10
```
{% endcode-tabs-item %}
{% endcode-tabs %}

