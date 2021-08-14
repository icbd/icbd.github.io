---
layout: post
title:  PostgreSQL 外键约束
Author: CBD
tags:   [PostgreSQL, ForeignKey]
---

首先推荐阅读 [为什么数据库不应该使用外键](https://draveness.me/whys-the-design-database-foreign-key)

我们以 user 和 score 为例, ProstgreSQL 12.6 为试验环境:

```yml
users:
  - id
  - name

scores:
  - id
  - user_id
  - level
```

通常情况下, 我们会为 `users` 和 `scores` 分别创建主键 `id`, `user_id` 做为 `scores` 的外键来保障引用完整性(Referential Integrity).

```sql
CREATE TABLE "public"."test_users" (
  "id" int8 NOT NULL,
  "name" varchar(255) COLLATE "pg_catalog"."default",
  CONSTRAINT "test_users_pkey" PRIMARY KEY ("id")
)
;

CREATE TABLE "public"."test_scores" (
  "id" int8 NOT NULL,
  "user_id" int8 NOT NULL,
  "level" varchar(255) COLLATE "pg_catalog"."default",
  CONSTRAINT "test_scores_pkey" PRIMARY KEY ("id"),
  CONSTRAINT "test_scores_user_id_fkey" FOREIGN KEY ("user_id") REFERENCES "public"."test_users" ("id") ON DELETE CASCADE ON UPDATE CASCADE
)
;
```

以上的设计有这么几个特点:

1. 所有表的主键都以 `id` 命名;
2. 外键指向关联表的主键字段, 并以 `xxx_id` 的方式命名;
3. 外键设置为 NOT NULL;
4. 设置级联删除和更新.

这是通常的设计, 也是稳妥的设计, 这四条是良好风格的约定, 但都不是强制的要求.

那下面我们就打破这些约定, 来看看外键本身的特点.

## 主键

1. 一个表至多有一个主键
2. 一个表可以没有主键(但没有理由不设置主键)
3. `PRIMARY KEY` 即 `UNIQUE NOT NULL`
4. 主键可以由一个字段或一组字段构成
5. PG 自动为主键添加 unique B-Tree 索引

## 外键

1. 指向的关联表字段必须为 unique (可以为 NULL)
2. 当外键字段为 NULL 时, 外键约束会失效
3. 当多字段外键中存在字段为 NULL 时, 外键约束会失效
4. PG 不会自动给外键加索引
5. 外键可设置被更新/删除后的操作

on update && on delete

* no action (默认)
* restrict
* cascade
* set null
* set default

`restrict` 和 `no action` 最终效果是一样的, 区别是在事务中的生效阶段不同.

> The essential difference between these two choices is that NO ACTION allows the check to be deferred until later in the transaction, whereas RESTRICT does not.

综上, 如果没有按照常规约定来设计外键, 务必要把外键设置为 NOT NULL, 必要时添加索引.
