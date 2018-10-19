---
description: Database Management
---

# 数据库管理

## 对表的管理

### 创建表

这里有一个关于创建 "users" 表的例子:

```sql
create table users (
    id serial primary key, -- id 自动递增
    name character varying, -- 指定字符串输出长度大小
    preferences jsonb, -- 字段类型为JSON非常适合存储非结构化的数据
    created_at timestamp without time zone -- 始终以UTC格式存储时间
);
```

你也有机会指定非空约束和默认值：

```sql
create table users (
    id serial primary key,
    name character varying not null,
    active boolean default true
);
```



### 删除表

```sql
drop table funky_users;
```



### 重命名表

```sql
alter table events rename to events_backup;
```



### 清空表

要非常小心这段代码，它会清空 PosgreSQL 表中的所有内容，在开发中很有用处，但是它很少想在生产环境中应用。

```sql
truncate my_table;
```

如果你又一个序列ID列，并且你想重启它的序列（既重启ID重1开始）

```sql
truncate my_table restart identity;
```



### 复制表

有时候表的复制对你很有用：

```sql
create table dupe_users as (select * from users);

-- 这个 'with no data' 意思是只有结构，没有实际的行
create table dupe_users as (select * from users) with no data;
```



### 添加列

这里有一个例子关于在 users 表中添加 created\_at 时间戳列：

```sql
alter table users add column created_at timestamp without time zone;
```

添加一个 String \( varchar \) 类型并且设置非空约束的列：

```sql
alter table users add column bio string character varying not null;
```

添加一个 Boolean 类型并且设置默认值的列：

```sql
alter table users add column active boolean default true;
```



