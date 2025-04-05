---
title: Generics and Functional Interfaces in Java
published: 2025-04-04
tags: [Java]
category: JavaKnowledgeCache
description: Guide of Generics and Functional Interfaces
draft: false
---

## 📘 Java 泛型与函数式接口：以缓存工具类为例

本文通过 `CacheClient` 类，讲解 Java 中泛型 `R` 和函数式接口 `Function` 的实际用法。

------

### 🔁 泛型 `R` 与 `ID` 是什么？

```java
public <R, ID> R queryWithPassThrough(
    Long time, TimeUnit unit,
    String keyPrefix, ID id,
    Class<R> type,
    Function<ID, R> dbFallback
)
```

- `R`：**返回值类型**，比如 Shop、User、Blog
- `ID`：**主键类型**，一般是 Long 或 String
- 这个方法是通用的，可以处理不同类型的缓存数据。

**例子：**

```java
cacheClient.queryWithPassThrough(
    30L, TimeUnit.MINUTES,
    "shop:", 1L,
    Shop.class,
    shopService::getById
);
```

| 泛型 | 实际类型        |
| ---- | --------------- |
| `R`  | `Shop`          |
| `ID` | `Long`          |
| `r`  | `Shop` 类型对象 |

------

### ⚙️ 函数式接口 `Function<ID, R>` 是什么？

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

你传入的是一个“查数据库”的函数，比如：

```java
shopService::getById
```

这个函数的作用是：

> 给我一个 `ID`（比如 1L），我返回一个 `R`（比如一个 Shop 对象）

在代码中：

```java
R r = dbFallback.apply(id);
```

就相当于：

```java
Shop r = shopService.getById(id);
```

------

### ✅ 使用函数式接口的好处

- 🔄 **逻辑复用**：不管是 Shop、User 还是 Blog 都可以用一个方法处理
- 🔓 **解耦合**：`CacheClient` 不依赖具体 Service
- 🧹 **更整洁**：调用时更短、更直观（支持 lambda 和方法引用）

------

### 🔄 总结对照表

| 名称                   | 类型              | 说明                          |
| ---------------------- | ----------------- | ----------------------------- |
| `R`                    | 泛型              | 表示返回的数据类型            |
| `ID`                   | 泛型              | 表示主键 ID 的类型            |
| `Function<ID, R>`      | 函数式接口        | 接收一个 ID，返回一个数据对象 |
| `dbFallback`           | 方法引用或 Lambda | 实际查数据库的逻辑            |
| `dbFallback.apply(id)` | 方法调用          | 返回从数据库查到的数据        |

------

### 🌟 Bonus：你也可以用 Lambda 写

```java
cacheClient.queryWithPassThrough(
    30L, TimeUnit.MINUTES,
    "shop:", 1L,
    Shop.class,
    id -> shopService.getById(id)
);
```

更简洁了有没有～😎

