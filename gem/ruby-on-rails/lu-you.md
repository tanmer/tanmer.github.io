# 路由

## 配置前后端分离

当我们快速启动一个Rails项目时，rails g scaffold是最好的工具，马上就能生成CRUD页面。但是，CRUD功能往往都是管理端需要，前端最多的还是Index和Show页面。为了实现前后端代码分离，我们把所有前端controller/view放到frontend目录，rails g scaffold生成的作为管理端。配置routes.rb的代码如下：

![](../../.gitbook/assets/image%20%2834%29.png)

