---
title: JavaKnowledgeCache
published: 2025-04-03
tags: [Java]
description: A note I wrote after looking something up during my project
category: JavaKnowledgeCache
draft: false
---

# JavaKnowledgeCache

## SpringMVC 中 @GetMapping 和 @PathVariable 的用法详解

### @GetMapping("/{id}") 是 Java 自带的吗？

不是 Java 自带的，是 Spring Framework 提供的注解，准确来说是 SpringMVC 里的。

#### 所属包：

```java
import org.springframework.web.bind.annotation.GetMapping;
```

它是对 `@RequestMapping(method = RequestMethod.GET)` 的简化写法，用来表示这是一个处理 **HTTP GET 请求** 的方法。

---

### 注解怎么用？

注解是 Java 里的一种特殊语法结构，用 `@注解名` 表示，它可以放在类、方法、参数、字段上面，表示一些“元信息”。

在 SpringMVC 中，注解是用来告诉框架 **这个方法是用来干什么的**。

#### 示例：

```java
@GetMapping("/user/{id}")
public String getUser(@PathVariable("id") Long id) {
    // 根据 id 获取用户
    return "用户信息";
}
```

这个意思就是：当访问 `/user/123` 时，就执行这个方法，`id = 123`。

---

### @PathVariable("id") 是什么？Java 里有这个东西吗？

不是 Java 标准库自带的，是 SpringMVC 提供的注解，用来从 URL 路径里提取参数。

#### 所属包：

```java
import org.springframework.web.bind.annotation.PathVariable;
```

#### 用法说明：

假设有请求路径：

```
GET /shop/5
```

你写的控制器方法是：

```java
@GetMapping("/shop/{id}")
public Result queryShopById(@PathVariable("id") Long id) {
    ...
}
```

那么 SpringMVC 就会：

- 自动把 URL 中的 `5` 提取出来
- 赋值给 `Long id` 参数
- 然后你就可以在方法中使用 `id`

## SpringBoot 三层架构 + Redis 缓存注入与调用流程详解

### 1. 为什么要使用三层架构（Controller → Service → Mapper）？

使用三层架构的目的：

- **解耦合**：Controller 只负责接收请求和返回结果，业务逻辑放在 Service。
- **好维护**：业务逻辑变了，只需要改 Service，不影响 Controller。
- **好扩展**：如果要换实现类，只需改注入类即可。
- **易测试**：可以 mock 掉 Service 层，单独测试 Controller 或 Service。

示意流程图：

```
浏览器请求 /shop/1
         ↓
@Controller
public Result queryShopById(id) {
    return shopService.queryById(id);
}
         ↓
@Service
public Result queryById(id) {
    // 查缓存 → 查数据库 → 写缓存 → 返回
}
         ↓
DAO（继承自 ServiceImpl）
调用 MyBatis-Plus 的 getById()
         ↓
数据库
```

---

### 2. 为什么要用接口 `IShopService` 和实现类 `ShopServiceImpl`？

#### 作用：

- Spring 使用接口+实现类进行依赖注入管理。
- 接口定义功能（方法签名），实现类定义怎么做。
- 更利于未来扩展、替换或 mock 测试。

---

### 3. Bean 是什么？

#### JavaBean（原始 Java）：

- 有无参构造器
- 私有成员变量
- 公共 getter 和 setter

#### SpringBean：

- 被 Spring 容器管理的对象（通过注解或配置生成）
- 生命周期由 Spring 管理，自动注入和使用

#### 常见 Bean 注解：

| 注解                              | 作用       |
| --------------------------------- | ---------- |
| `@Component`                      | 基础 Bean  |
| `@Service`                        | 服务类     |
| `@Repository`                     | 数据访问层 |
| `@Controller` / `@RestController` | 控制层     |

---

### 4. @Resource 是必须的吗？什么时候用？

#### ✅ 不是必须，但如果你想让 Spring 注入一个对象，必须用依赖注入注解。

常见的两种方式：

| 注解         | 来源    | 说明               |
| ------------ | ------- | ------------------ |
| `@Autowired` | Spring  | 更灵活，按类型注入 |
| `@Resource`  | JDK标准 | 更简洁，按名称注入 |

#### 判断标准：

- **不是你 new 的对象**，而是想让 Spring 自动帮你生成的 → 就要加 `@Resource` 或 `@Autowired`

---

### 5. Bean 的创建流程（以 `StringRedisTemplate` 为例）

```java
@Bean
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory factory) {
    return new StringRedisTemplate(factory);
}
```

Spring 启动时：

1. 调用该方法生成对象
2. 存入 IOC 容器（Bean 工厂）
3. 你写 `@Resource` 注入时，不会重新创建，而是复用这个对象

#### Bean 是单例的：

```java
@Scope("singleton") // 默认就是这个
```

多个地方注入，只是引用同一个对象。

---

### 6. StringRedisTemplate 是干嘛的？

#### 作用：

Spring 封装的 Redis 客户端工具类，操作 **字符串类型**的 Redis 数据最常用。

#### 常用方法：

| 功能         | 方法                  | 示例         |
| ------------ | --------------------- | ------------ |
| 写入         | `opsForValue().set()` | 保存缓存     |
| 读取         | `opsForValue().get()` | 读取缓存     |
| 设置过期时间 | `expire()`            | 过期机制     |
| 删除键       | `delete()`            | 删除缓存     |
| 判断存在     | `hasKey()`            | 判断缓存命中 |

#### 和 RedisTemplate 区别：

| 类名                            | 用途         | 支持的数据类型 |
| ------------------------------- | ------------ | -------------- |
| `RedisTemplate<Object, Object>` | 通用，但复杂 | 所有类型       |
| `StringRedisTemplate`           | 简单易用     | 仅字符串       |

---

### 7. 多个 Service 使用一个 `StringRedisTemplate` 会冲突吗？

#### ✅ 不会冲突！

- 这个对象是一个客户端连接，多个类共用没有问题。
- 每次使用都是独立的 Redis 请求。
- Spring 会用连接池管理，多线程安全。

唯一会“冲突”的情况是：

- 多个类写入相同的 Redis key（如 `"shop:1"`），值会被覆盖

- 所以实际开发中我们会做 key 的前缀或业务隔离，比如：

  ```java
  stringRedisTemplate.set("shop:" + shopId, jsonStr);
  ```

---

### 8. JSONUtil.toBean 是什么？

#### 场景：

从 Redis 取出的数据是字符串，要转换成 Java 对象。

#### 用法：

```java
Shop shop = JSONUtil.toBean(shopJson, Shop.class);
```

- `shopJson` 是 JSON 字符串
- `Shop.class` 是目标对象类型
- 这就完成了 JSON → Java 对象的反序列化

