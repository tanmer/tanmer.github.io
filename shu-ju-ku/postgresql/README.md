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

### 更改数据表的拥有者

```sql
SELECT 'ALTER TABLE '|| schemaname || '.' || tablename ||' OWNER TO new-owner;'
FROM pg_tables WHERE NOT schemaname IN ('pg_catalog', 'information_schema')
ORDER BY schemaname, tablename;

SELECT 'ALTER SEQUENCE '|| sequence_schema || '.' || sequence_name ||' OWNER TO new-owner;'
FROM information_schema.sequences WHERE NOT sequence_schema IN ('pg_catalog', 'information_schema')
ORDER BY sequence_schema, sequence_name;
```

### 清空数据库

执行下面查询，生成对所有表`drop`的SQL，然后复制、粘贴、执行。

```sql
select 'drop table "' || tablename || '" cascade;' as drop_table from pg_tables where schemaname = 'public';
```

 

