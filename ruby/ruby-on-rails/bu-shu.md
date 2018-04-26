# 部署

## 部署中遇到的问题

#### 通过capistrano部署代码时，报UglifyJs错误

这个问题，通常是因为UglifyJs不支持ES2015语法，而一些npm包中又有这种新语法，导致编译失败。

临时解决办法：删除UglifyJs插件

{% code-tabs %}
{% code-tabs-item title="config/webpack/production.js" %}
```javascript
const environment = require('./environment')
environment.plugins.delete("UglifyJs")
module.exports = environment.toWebpackConfig()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

**或者：**

添加`babel-present-stage-2`, 关闭uglify

```bash
yarn add babel-present-stage-2
```

配置`.babelrc`

```javascript
{
  "presets": [
    ["env", {
      "modules": false,
      "targets": {
        "browsers": "> 1%",
        "uglify": false
      },
      "useBuiltIns": true
    }],
    "stage-2"
  ],
  ...
}
```



