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

> 用来对表中数据做筛选

查询 &lt;表名&gt; 所有列信息:

```text
select * from <表名>;   <=>    rails: Table_name.all
```

查询 &lt;表名&gt; 中 &lt;列名1&gt;、&lt;列名2&gt; 的所有内容

```text
select <列名1>, <列名2> from <表名>;
```

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



