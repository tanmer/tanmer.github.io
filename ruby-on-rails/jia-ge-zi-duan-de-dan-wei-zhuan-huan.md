# 价格字段的单位转换

我们在存储一个产品的价格时，字段类型的设计可以有一下几种：

* Decimal - 以元为单位存储浮点数
* Integer - 以分为单位储存整数

如果用浮点数存储数据，那么在我们表单录入价格时，时常会遇到一个奇怪的问题，我录入的价格是`9.12`元，而存储数据库时莫名其妙地变成了`911.999999999999` ，然后再呈现到前段，就很怪异了。

在ruby中我可以通过下面测试，发现同样问题：

```ruby
2.3.4 :017 > "9.12".to_f * 100
 => 911.9999999999999
```

因此我们就有了以分为单位存储的想法，但是，以分为单位，我们在生成表单时，又想让用户以元为单位录入价格，这个转换就显得有些麻烦了，我们可能会这么做

```bash
rails g scaffold product name price_cents:integer
```

{% code-tabs %}
{% code-tabs-item title="app/models/product.rb" %}
```ruby
class Product < ApplicationRecord
    # 数据库存储价格字段是 price_cents，整形，单位为分
    def price
        (price_cents || 0) / 100.0
    end

    def price=(v)
        # 这里用.round就是为了解决上面的911.99999999问题
        self.price_cents = (v.to_f * 100).round
    end
end
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="app/controllers/products\_controller.rb" %}
```ruby
def product_params
    params.require(:product).permit(:name, :price)
end
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="app/views/products/\_form.html.erb" %}
```markup
  <div class="field">
    <%= form.label :price %>
    <%= form.text_field :price, id: :product_price %>
  </div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 封装一下代码，做到足够DRY

{% code-tabs %}
{% code-tabs-item title="app/models/application\_record.rb" %}
```ruby
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true

  class << self
    def price_attr(model_attr, db_attr="#{model_attr}_cents")
      class_eval <<-CODE, __FILE__, __LINE__ + 1
        def #{model_attr}
          (#{db_attr} || 0) / 100.0
        end

        def #{model_attr}=(v)
          self.#{db_attr} = (v.to_f * 100).round
        end
      CODE
    end
  end
end
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="app/models/product.rb" %}
```ruby
class Product < ApplicationRecord
    price_attr :price
end
```
{% endcode-tabs-item %}
{% endcode-tabs %}

以上代码，足够应付我们的价格前后端单位转换问题。如果需要更高级额功能，我们可以尝试Gem [money](https://github.com/RubyMoney/money) 或 [money-rails](https://github.com/RubyMoney/money-rails)

