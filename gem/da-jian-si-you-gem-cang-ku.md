# 搭建私有Gem仓库

Gem私有仓库，我们用`Geminabox`快速搭建[https://github.com/geminabox/geminabox](https://github.com/geminabox/geminabox)

{% code-tabs %}
{% code-tabs-item title="config.ru" %}
```ruby
#
# This is a simple rackup file for geminabox. It allows simple role-based authorization.
#
# roles:
# - developer
# - upload
# - delete
# - admin (can do anything)
#
# For example, a developer who can access the service and upload new gems would have the following roles: `%w(developer upload)
#

require "rubygems"
require "geminabox"

Geminabox.rubygems_proxy = false
Geminabox.data = "/geminabox/data"

API_KEYS = {
  ENV['DEVELOPER_API_KEY'] => { password: '', roles: %w(developer) },
  ENV['ADMIN_API_KEY'] => { password: '', roles: %w(admin) }
}

use Rack::Session::Pool, expire_after: 1000 # sec
use Rack::Protection

Geminabox::Server.helpers do
  def protect!(role='developer')
    unless has_role?(role)
      response['WWW-Authenticate'] = %(Basic realm="Gem In a Box")
      halt 401, "Not Authorized.\n"
    end
  end

  def auth
    @auth ||= Rack::Auth::Basic::Request.new(request.env)
  end

  def username
    auth ? auth.credentials.first : nil
  end

  def password
    auth ? auth.credentials.last : nil
  end

  def user_roles
    API_KEYS[username][:roles]
  end

  def authenticated?
    return false unless auth.provided? && auth.basic? && auth.credentials
    api_key = API_KEYS[username]
    !api_key.nil? && password == api_key[:password]
  end

  def current_user_roles
    authenticated? ? user_roles : []
  end

  def has_role?(role)
    current_user_roles.include?('admin') || current_user_roles.include?(role)
  end
end

Geminabox::Server.before '/upload' do
  protect!('upload')
end

Geminabox::Server.before do
  if request.delete?
    protect!('delete')
  else
    protect!('developer')
  end
end

Geminabox::Server.before '/api/v1/gems' do
  unless env['HTTP_AUTHORIZATION'] == 'API_KEY'
    halt 401, "Access Denied. Api_key invalid or missing.\n"
  end
end

run Geminabox::Server

```
{% endcode-tabs-item %}
{% endcode-tabs %}

把上面内容保存到`config.ru`文件，然后运行`ADMIN_API_KEY=admin rackup`即可启动服务，登录用户名是`admin`，密码填空

