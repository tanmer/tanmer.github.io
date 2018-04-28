# Git

## Git flow开发流程

### 安装Git flow

```bash
curl -L https://raw.githubusercontent.com/nvie/gitflow/develop/contrib/gitflow-installer.sh -o /tmp/gitflow-installer.sh
sudo bash /tmp/gitflow-installer.sh
```

### 初始化

开始规定工作流，进入`develop`分支开发

```bash
git flow init -d
```

### 开始一个Feature

```bash
git flow feature start <your feature>
```

完成一个Feature

```bash
git flow feature finish <your feature>
```

上传一个 Feature

```bash
git flow feature publish <name>
```

或上传一个Feature到某个remote

```bash
git flow feature pull <remote> <name>
```

### 开始一个Release分支

```bash
git flow release
git flow release start <release> [<base>]
git flow release finish <release>
```

### 开始一个Hotfix分支

```bash
git flow hotfix
git flow hotfix start <release> [<base>]
git flow hotfix finish <release>
```



