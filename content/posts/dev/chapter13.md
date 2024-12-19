---
title: "树结构工具类封装"
date: 2024-10-01T10:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 介绍

树结构工具类封装

## 1 开始

### 1.1 可树化接口

```
/**
 * 可树化
 */
public interface Treeable<T, K> {
    /**
     * 获取 id
     */
    K getId();

    /**
     * 获取父级 id
     */
    K getParentId();

    /**
     * 获取子节点
     */
    List<T> getChildren();

    /**
     * 设置子节点
     */
    void setChildren(List<T> children);
}
```

### 1.2 递归建树

```
public static <T extends Treeable<T, K>, K> List<T> build(List<T> list, K pid) {
    if (list == null || list.isEmpty()) {
        return null;
    }
    List<T> tree = new ArrayList<>();
    list.forEach(item -> {
        if (pid.equals(item.getParentId())) {
            item.setChildren(build(list, item.getId()));
            tree.add(item);
        }
    });
    return tree;
}
```

### 1.3 直接建树

```
public static <T extends Treeable<T, K>, K> List<T> build(List<T> list) {
    Map<K, T> itemMap = new HashMap<>();
    List<T> tree = new ArrayList<>();
    list.forEach(item -> {
        T parent = itemMap.get(item.getParentId());
        if (parent != null) {
            List<T> children = parent.getChildren();
            if (children == null) {
                children = new ArrayList<>();
                parent.setChildren(children);
            }
            children.add(item);
        } else {
            tree.add(item);
        }
        itemMap.put(item.getId(), item);
    });
    return tree;
}
```

## 2 使用

### 2.1 实体类实现 Treeable 接口

```
public class SysResourceVo extends SysResource implements Treeable<SysResourceVo, Integer> {
    private List<SysResourceVo> children;

    @Override
    public List<SysResourceVo> getChildren() {
        return children;
    }

    @Override
    public void setChildren(List<SysResourceVo> children) {
        this.children = children;
    }
}
```

### 2.2 构建树结构

```
/**
  * 菜单树
  */
public static List<SysResourceVo> buildMenuTree(List<SysResourceVo> resources) {
    return TreeUtil.build(resources);
}
```