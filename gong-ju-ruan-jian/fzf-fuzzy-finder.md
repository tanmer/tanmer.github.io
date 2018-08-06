# fzf\(Fuzzy Finder\)

[https://github.com/junegunn/fzf](https://github.com/junegunn/fzf)

这是一个超快的文件路径模糊搜索工具，功能就像`Sublime Text`的`ctrl+p`,输入`acacrb`就能搜索出文件`app/controller/application_controller.rb`

## 安装

```bash
brew install fzf
```

## Vim支持fzf

{% code-tabs %}
{% code-tabs-item title="~/.vimrc" %}
```text
set rtp+=/usr/local/opt/fzf
" 设置快捷键\+f开启搜索
map <Leader>f :FZF<CR>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

![](../.gitbook/assets/image%20%2831%29.png)

