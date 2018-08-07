---
description: 取代Mac自带的Spotlight search
---

# Alfred

## 自动通过翻墙浏览器上网

![](../.gitbook/assets/image%20%2813%29.png)

### 原理

通过Chrome浏览器的`Profile`功能，专门新建一个`翻墙`Profile，配置好翻墙代理服务器。Alfred通过命令行打开Chrome浏览器，传入参数，让Chrome选择对应的Profile，打开网站。

{% code-tabs %}
{% code-tabs-item title="打开Chrome浏览器，指定 Profile的命令行" %}
```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" "http://www.google.com" --profile-directory="Profile 4"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

这里唯一的问题就是`--profile-directory`填什么。这里我们可以借助 [Alfred Chrome](https://github.com/ShogunPanda/alfred-chrome) 得到浏览器Profile对应的目录是什么

### 安装Alfred Chrome Workflow

打开 [http://www.packal.org/workflow/alfred-chrome](http://www.packal.org/workflow/alfred-chrome)

下载 [https://github.com/packal/repository/raw/master/it.cowtech.alfred.chrome/alfred\_chrome.alfredworkflow](https://github.com/packal/repository/raw/master/it.cowtech.alfred.chrome/alfred_chrome.alfredworkflow)

双击下载的文件，workflow会自动添加到Alfred

![](../.gitbook/assets/image%20%281%29.png)

双击cr

![](../.gitbook/assets/image%20%2835%29.png)

点下面圆圈处，会打开Finder

![](../.gitbook/assets/image%20%286%29.png)

### 找出翻墙Profile所在的目录名

在bash中执行alfred-chrome文件，会输出JSON文件，从中找出自己配置的翻墙profile的目录名

![](../.gitbook/assets/image%20%284%29.png)

这里，我的翻墙profile目录是`Profile 4`

下载我制作好的workflow文件 [gfw.alfredworkflow](https://github.com/tanmer/tanmer.github.io/raw/master/.gitbook/assets/gfw.alfredworkflow) , 打开workflows, 双击`/bin/bash`节点，修改`Profile 4`为你自己的翻墙目录

```text
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" $1 --profile-directory="Profile 4"
```

### 开始使用

Alfred搜索条中试试输入`fq`,`gem`,`github`,`superu`,`stackof`

