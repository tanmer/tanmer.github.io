# 深度冻结变量 Deep Freeze

Ruby自带的`freeze`无法深度冻结，如：

```ruby
$ irb
2.4.4 :001 > h={a: [1,2], b: { c: [2,3]}, c: 'hello'}.freeze
 => {:a=>[1, 2], :b=>{:c=>[2, 3]}, :c=>"hello"}
2.4.4 :002 > h[:c] << ' there'
 => "hello there"
2.4.4 :003 > h
 => {:a=>[1, 2], :b=>{:c=>[2, 3]}, :c=>"hello there"}
```

通过下面扩展，可以实现深度冻结：

```ruby
# Recursively freeze self if it's Enumerable
# Supports all Ruby versions 1.8.* to 2.2.*+
module Kernel
  alias deep_freeze freeze
  alias deep_frozen? frozen?
end

module Enumerable
  def deep_freeze
    if !@deep_frozen
      each(&:deep_freeze)
      @deep_frozen = true
    end
    freeze
  end

  def deep_frozen?
    !!@deep_frozen
  end
end
```

```ruby
2.4.4 :021 > h={a: [1,2], b: { c: [2,3]}, c: 'hello'}.deep_freeze
 => {:a=>[1, 2], :b=>{:c=>[2, 3]}, :c=>"hello"}
2.4.4 :022 > h[:c] << ' there'
RuntimeError: can't modify frozen String
	from (irb):22
	from /Users/xiaohui/.rvm/rubies/ruby-2.4.4/bin/irb:11:in `<main>'
```



