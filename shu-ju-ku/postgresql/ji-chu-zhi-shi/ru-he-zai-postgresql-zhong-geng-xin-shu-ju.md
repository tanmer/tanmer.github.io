---
description: How to Update in PostgreSQL
---

# PostgreSQL中更新数据

```text
-- 更新 users 表中列名为 updated_at 的所有行数据
update users set updated_at = now();

-- 更新 users 表中列名为 updated_at 条件为 id = 1 这一行数据
update users set updated_at = now() where id = 1;

```

