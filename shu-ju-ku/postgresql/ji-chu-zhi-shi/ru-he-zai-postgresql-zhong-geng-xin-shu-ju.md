---
description: How to Update in PostgreSQL
---

# 如何在PostgreSQL中更新数据

```text
-- 更新列名为 updated_at 的所有行数据
update users set updated_at = now();

-- 更新列名为 updated_at 并且 id = 1 这一行数据
update users set updated_at = now() where id = 1;

```

