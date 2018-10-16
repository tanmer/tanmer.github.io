# 代码规范

对于编程的代码规范，我们都遵循国际习惯，做到规范统一，每个程序员都能看懂他人的代码。

* [HTML & CSS](https://google.github.io/styleguide/htmlcssguide.html)
* [Javascript](https://github.com/airbnb/javascript)
* [Ruby](https://github.com/JuanitoFatas/ruby-style-guide/blob/master/README-zhCN.md)
  * 使用Ruby 2.3+
  * 使用`dig`方法深度获取Hash的值
    * `params[:search][:keywords]` 不推荐
    * `params.dig(:search, :keywords)` 推荐
* [Ruby On Rails](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhCN.md)
  * Assets资源引用途径的优先级：[RubyGems](https://rubygems.org/gems/bootstrap) &gt; [RailsAssets](https://rails-assets.org/#/components/bootstrap) &gt; [NPM](https://www.npmjs.com/package/bootstrap) &gt; Download

