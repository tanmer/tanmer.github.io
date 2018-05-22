# PostgreSQL

### 创建用户

```sql
create role xiaohui with login password 'this-is-a-password';
```

### 创建数据库

```sql
create database project;
```

### 设置数据库拥有者

```sql
alter database project owner to xiaohui; 
grant all privileges on database project to xiaohui;
```

### 清空数据库

执行下面查询，生成对所有表`drop`的SQL，然后复制、粘贴、执行。

```sql
select 'drop table "' || tablename || '" cascade;' as drop_table from pg_tables where schemaname = 'public';
```

 

