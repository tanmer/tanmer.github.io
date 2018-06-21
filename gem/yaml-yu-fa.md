# YAML语法

## 数据继承

```ruby
require 'yaml'
y = YAML.load(<<-STR)
.base: &base "Tanmer Inc."
dev:
  name: *base
STR
# => {".base"=>"Tanmer Inc.", "dev"=>{"name"=>"Tanmer Inc."}}
```



