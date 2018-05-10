# PostgreSQL

## 清空数据库

执行下面查询，生成对所有表`drop`的SQL，然后复制、粘贴、执行。

```sql
select 'drop table "' || tablename || '" cascade;' as drop_table from pg_tables where schemaname = 'public';
```

 

