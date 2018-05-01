---
description: 'Learn to create , select and delete database in PostgreSQL'
---

# Create , Select , Delete

> 登录 PostgreSQL 控制台

```text
psql postgres
```

## Create

> 创建数据库

创建:

```text
create database test;
```

显示所有数据库列表:

```text
\l
\?        # 显示帮助说明
```

连接 test 数据库:

```text
\c test;
```

## Select

## Delete

> 用来删除行

```text
delete from <表名> where <列名> = <值>;
```

在不删除表的情况下, 删除所有行:

```text
delete from <表名>;
or
delete * from <表名>;
```



