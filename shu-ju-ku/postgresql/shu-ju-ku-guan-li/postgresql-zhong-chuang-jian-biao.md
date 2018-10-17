---
description: How to Create a Table in PostgreSQL
---

# PostgreSQL中创建表

#### 这里有一个关于创建 "users" 表的例子:

```sql
create table users (
    id serial primary key, -- id 自动递增
    name character varying, -- 指定字符串输出长度大小
    preferences jsonb, -- 字段类型为JSON非常适合存储非结构化的数据
    created_at timestamp without time zone -- 始终以UTC格式存储时间
);
```

#### 你也有机会指定非空约束和默认值：

```sql
create table users (
    id serial primary key,
    name character varying not null,
    active boolean default true
);
```

