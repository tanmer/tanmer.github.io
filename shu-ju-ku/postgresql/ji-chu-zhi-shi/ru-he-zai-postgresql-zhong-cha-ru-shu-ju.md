# PostgreSQL中插入数据

#### 这是Postgresql最短，最简单插入方法，你只需要按顺序的插入指定的值，所以，如果你有10列，你必须指定10个值。

```text
-- 假设这里有个 "users" 表只有3列：: first_name, last_name, email, 并且按此顺序排列
users values ('John', 'Doe', 'john@dow.com');
```

#### 如果你有很多列，当时你想指定其中某一列：

```text
insert into users (first_name) values ('John');
```

#### 如果想在列中插入JSON数据，只需要将JSON数据包裹在单引号字符串中。

```text
insert into users (preferences) values ('{ "beta": true }')
```

#### 如果插入的数据违背了一个特定的约束，这是你可以使用 Postgres 在冲突子句中指定当发生这种情况时该怎么做，例如：想象你有一个webhook系统，你想优雅的处理webhook中重复的事情。

```text
-- 如果我们已经记录（捕获）到 webhook 的冲突，就什么都不要做。
insert into stripe_webhooks (event_id)
values ('evt_123')
on conflict do nothing;
```

#### 你也可以在冲突的时候执行 "upserts"   \(更新或插入\)数据。

```text
-- 如果你有一个唯一的email索引
insert into users (email, name)
value ('john@dow.com', 'Jane Doe')
on conflict (email) do update set name = excluded.name; -- excluded.name 指的是 'Jane Doe'
```

