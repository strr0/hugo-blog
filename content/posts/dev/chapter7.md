---
title: "MySql 树形查询"
date: 2023-11-29T10:00:00+08:00
categories: ["development"]
tags: ["mysql"]
draft: false
---

## 介绍

MySql 树形查询

## 1 示例

### 1.1 查询所有子节点

```
select id from(
    select id, parent_id, if(find_in_set(parent_id, @pids) > 0, @pids := concat(@pids, ',', id), 0) ischild
    from (select id, parent_id from ${treeTableName} order by parent_id, id) t1, (select @pids := #{pid}) t2) t3
where ischild != 0
```

### 1.2 查询所有父节点

```
select t4.* from (
    select @r _id, (select @r := parent_id from ${treeTableName} where id = _id) _pid, @l := @l + 1 _l
    from (select @r := #{id}, @l := 0) t1, ${treeTableName} t2) t3 left join ${treeTableName} t4 on t3._id = t4.id
where t3._pid is not null
```