# Bash Shell

## 通用

### 如何改变指定用户的登录shell

```bash
chsh <用户名> -s <新shell>
chsh xiaohui -s /bin/sh
```

### 标准输出和错误输出同时重定向到同一位置

方法一：

```bash
2>&1 (如# ls /usr/share/doc > out.txt 2>&1 )
```

方法二：

```bash
&> (如# ls /usr/share/doc &> out.txt )
```

## 条件判断

```bash
if [ 条件 ]
then
命令1
命令2
…..
else
if [ 条件 ]
then
命令1
命令2
….
else
命令1
命令2
…..
fi
fi
```

### 比较两个数字

```bash
#!/bin/bash
x=10
y=20
if [ $x -gt $y ]
then
echo "x is greater than y"
else
echo "y is greater than x"
fi
```

### 如何测试文件

```text
Test         用法
-d 文件名    如果文件存在并且是目录，返回true
-e 文件名    如果文件存在，返回true
-f 文件名    如果文件存在并且是普通文件，返回true
-r 文件名    如果文件存在并可读，返回true
-s 文件名    如果文件存在并且不为空，返回true
-w 文件名    如果文件存在并可写，返回true
-x 文件名    如果文件存在并可执行，返回true
```

比如测试文件是否存在：

```bash
printf "正在复制 config/application.example.yml 到 config/application.yml ..."
if [ -e config/application.yml ]; then
  echo "已经存在，跳过复制"
else
  cp config/application{.example,}.yml
fi
```

### 





