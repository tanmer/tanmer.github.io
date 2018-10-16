# PostgreSQL

## 安装

### Docker安装

1. 我们的项目都会涉及到中文字符，会对中文字符按拼音排序，`Ubuntu`默认安装的`PostgreSQL`是`en_US.UTF-8`字符集排序，要支持拼音培训，需要改为`zh_CN.UTF-8`。
2. Mac下用`brew`安装的PostgreSQL字符集排序是`zn_CN.UTF-8`，但是很奇怪的是他并不能实现拼音排序，经研究之后确实找不到解决办法。 因为以上两点，所以我用这个镜像方便我们开发人员快速使用数据库

#### 如何启动

```bash
mkdir -p ~/postgresql/data
docker run -v $(realpath ~/postgresql/data):/var/lib/postgresql/data -p 5432:5432 --name postgresql-10 -d tanmer/postgresql:10
```

#### 进入psql CLI

```text
docker exec -it postgresql-10 psql -U postgres
```

#### 测试中文排序是否正确

```text
postgres=# select * from (values ('刘少奇'),('刘德华')) as a(c1) order by c1;
  c1
--------
刘德华
刘少奇
(2 rows)

postgres=#
```

#### 镜像 tanmer/postgresql:10 的 Dockerfile内容

```text
FROM postgres:10
MAINTAINER Xiaohui <xiaohui@tanmer.com>

RUN sed -i 's!deb.debian.org!mirrors.163.com!' /etc/apt/sources.list \
    && apt update \
    && apt install --reinstall locales \
    && echo zh_CN UTF-8 > /etc/locale.gen \
    && echo zh_CN.UTF-8 UTF-8 >> /etc/locale.gen \
    && echo en_US UTF-8 >> /etc/locale.gen \
    && echo en_US.UTF-8 UTF-8 >> /etc/locale.gen \
    && locale-gen

ENV LC_COLLATE zh_CN.UTF-8

```

## 使用

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

##  查询

### 获取某时区的时间

```sql
// 先获取所有的时区定义
select * from pg_timezone_names order by utc_offset
```

```text
               name               | abbrev | utc_offset | is_dst
----------------------------------+--------+------------+--------
 Africa/Abidjan                   | GMT    | 00:00:00   | f
 Africa/Accra                     | GMT    | 00:00:00   | f
 Africa/Addis_Ababa               | EAT    | 03:00:00   | f
 Africa/Algiers                   | CET    | 01:00:00   | f
 Africa/Asmara                    | EAT    | 03:00:00   | f
 Africa/Asmera                    | EAT    | 03:00:00   | f
 Africa/Bamako                    | GMT    | 00:00:00   | f
```

```sql
// 把字段created_at的UTC时间转换为用户设定的本地时间
select created_at, timezone, created_at at time zone 'UTC' at time zone timezone as localtime from users;
```

```text
         created_at         |   timezone    |         localtime
----------------------------+---------------+----------------------------
 2018-10-10 03:36:33.686459 | Asia/Shanghai | 2018-10-10 11:36:33.686459
 2018-10-10 04:11:23.707863 | PST           | 2018-10-09 20:11:23.707863
(2 rows)
```



