---
description: 根据设备类型自动渲染View页面
---

# 根据设备类型自动渲染页面

### 添加Gem

```ruby
gem "browser"
```

### 定义方法

```ruby
class ApplicationController < ActionController::Base
  
  before_action :detect_device_variant

  def detect_device_variant
    # 定义手机端调用
    request.variant = :mobile if browser.device.mobile?
    # 定义平板，如果需要
    request.variant = :tablet if browser.device.tablet?
  end

end

```

### 配置View

找到需要配置的view

```ruby
application.html.erb  #原始模板文件
application.html+mobile.erb  #如果定义的有mobile
application.html+tablet.erb  #如果定义的有tablet
```

