---
title: Why Layered Architecture? The Difference Between UserService and Controller
published: 2025-04-02
description: Understanding the benefits of separating Service and Controller layers in Java Web development.
tags: [FullStack, FlashLife]
category: FlashLife (Java_Project)
draft: false
lang: zh
---

# 为什么要分层？——以 UserService 和 UserController 为例

在 Java Spring 开发中，将 UserService 和 UserController 分开是典型的“分层架构”设计，具有以下好处：

---

## 🧱 分层类比：餐厅点菜

| 角色 | 对应代码层 | 职责 |
|------|------------|------|
| 顾客（你） | 浏览器 / 前端 | 发起请求（我要点菜） |
| 服务员 | Controller（控制层） | 接收请求，传给后厨 |
| 厨师 | Service（服务层） | 真正干活（做菜） |
| 数据库 | 数据访问层 | 存放数据（食材） |

---

## ✅ 为什么要分开写？

### 1. 职责单一

- Controller 负责接收请求、返回响应。
- Service 负责处理核心业务逻辑。
- 职责清晰，代码更易于理解和维护。

---

### 2. 方便复用

- 一个 Service 方法可以被多个 Controller 或其他服务调用。
- 比如发送验证码功能，网页和 App 都能用。

---

### 3. 更容易测试

- 可以单独测试 UserService.sendCode() 方法。
- 不依赖前端或 Controller 层。

---

### 4. 更适合团队开发

- 团队成员可以各司其职。
- Controller 写接口，Service 写业务逻辑，互不干扰。

---

### 🤯 不分层的坏处

```java
@RestController
public class UserController {
    @GetMapping("/send")
    public String sendCode() {
        System.out.println("发送验证码...");
        // 还有很多逻辑都堆在这里
        return "验证码已发送";
    }
}
```

- 所有逻辑混在一起
- 不易测试，不易维护，不易扩展

---

## ✅ 总结表格

| 理由 | 解释 |
|------|------|
| 职责分明 | 每层干自己的事，结构清晰 |
| 易于复用 | Service 方法可以被多个接口调用 |
| 易于维护 | 改一处不影响其他 |
| 易于测试 | 每一层都能单独测试 |
| 符合开发规范 | 团队协作更容易 |

---

以上就是将 Service 与 Controller 分离的原因，它是构建大型、稳定、可维护系统的重要基础。

# 用户验证码发送流程（Java 后端分层逻辑总结）

本文件总结了 Shuchang 在开发 sendCode 接口时所走的标准流程，帮助理解为什么要这样做，以及每一步的作用。

---

## 🎯 目标

实现这样一个接口逻辑：

```java
@PostMapping("code")
public Result sendCode(@RequestParam("phone") String phone, HttpSession session) {
    return userService.sendCode(phone, session);
}
```

---

## 🧠 为什么要这样分步骤？

| 步骤 | 你做的事                                                  | 目的                                                 |
| ---- | --------------------------------------------------------- | ---------------------------------------------------- |
| 1    | 在 Controller 中写接口，调用 userService.sendCode(...)  | Controller 只负责接收请求，把业务交给 Service 层处理 |
| 2    | 接口 IUserService 中添加方法签名                        | 明确规定“这个服务应该提供 sendCode 方法”           |
| 3    | 找到 UserServiceImpl 实现类，使用快捷键自动生成实现方法 | 实际去写业务逻辑的地方                               |

---

## 🔁 整体结构和调用链

前端请求（POST /code）  
↓  
Controller（UserController）  
↓  
Service 接口（IUserService）  
↓  
Service 实现类（UserServiceImpl）  
↓  
返回 Result 给前端

---

## 📦 你创建的代码结构如下：

### 1. Controller 层

```java
@RestController
public class UserController {

    @Resource
    private IUserService userService;

    @PostMapping("code")
    public Result sendCode(@RequestParam("phone") String phone, HttpSession session) {
        return userService.sendCode(phone, session);
    }
}
```

---

### 2. Service 接口

```java
public interface IUserService extends IService<User> {
    Result sendCode(String phone, HttpSession session);
}
```

---

### 3. Service 实现类

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {

    @Override
    public Result sendCode(String phone, HttpSession session) {
        // TODO: 实现发送验证码逻辑
        return null;
    }
}
```

---

## ✅ 为什么这么做？

| 理由         | 说明                                             |
| ------------ | ------------------------------------------------ |
| 结构清晰     | 每层职责单一，Controller 不处理业务逻辑          |
| 易于扩展     | 未来可以复用 sendCode 方法给 App、小程序等使用 |
| 方便测试     | 可以单独测试 Service 层方法                      |
| 规范开发流程 | Java 开发团队通常都这样组织代码                  |
| 提高复用性   | Service 方法可以被多个地方复用                   |

# 登陆验证功能

![image-20250401224121656](src/content/posts/FlashLife/note1/images/image-20250401224121656.png)

## 🔒 什么是拦截器（Interceptor）？

拦截器是一种**在请求到达 Controller 前/后进行统一处理的机制**，属于 Spring MVC 的一个重要功能。

---

### ✅ 举个例子：

你访问 /user/me 时，Spring 会先看看有没有注册了拦截器，比如 LoginInterceptor。  
它会 **先执行 preHandle() 方法** —— 如果登录状态 OK，就放行；否则就拦截、返回 401。

---

### 📌 拦截器能干什么？

- 登录校验 ✅（你现在在用）
- 权限控制
- 日志记录
- 请求参数校验
- 全局设置请求参数 / Header 等

---

### ✅ 你项目中的拦截器核心逻辑：

```java
public boolean preHandle(...) {
    Object user = session.getAttribute("user");
    if (user == null){
        response.setStatus(401);
        return false; // 拦截
    }
    UserHolder.saveUser((UserDTO) user); // 保存登录信息
    return true; // 放行
}
```

---

## 🧠 为什么要用拦截器？

拦截器的 **最大优势是“统一处理，避免重复代码”**：

比如：

- 每个接口都要校验“用户是否登录”

如果不使用拦截器，你必须在每个 Controller 方法前写：

```java
if (session.getAttribute("user") == null) {
    return Result.fail("请先登录");
}
```

太麻烦 ❌，用拦截器一次搞定 ✅！

---

## 🧵 什么是 ThreadLocal？

ThreadLocal 是 Java 提供的一种 **线程内局部变量机制**。它的作用是：

> 在同一个线程中，任意位置都可以共享使用它设置的值；但线程之间互不干扰。

---

### ✅ 为什么我们用 ThreadLocal 来存用户？

拦截器放行后，请求就会继续进入 Controller、Service。

如果你想在后续逻辑中随时拿到“当前登录用户的信息”，又不想每次都写：

```java
UserDTO user = (UserDTO) session.getAttribute("user");
```

就可以使用一个工具类 UserHolder 来封装 ThreadLocal：

```java
public class UserHolder {
    private static final ThreadLocal<UserDTO> tl = new ThreadLocal<>();

    public static void saveUser(UserDTO user){
        tl.set(user);
    }

    public static UserDTO getUser(){
        return tl.get(); // 在任意地方取当前登录用户
    }

    public static void removeUser(){
        tl.remove(); // 请求结束后清理，避免内存泄漏
    }
}
```

> 这样你在 Service 或 Controller 中直接调用：UserHolder.getUser() 就能获取当前用户！简洁又优雅 🧼

---

## 🧩 配套关系一图流：

你发的图就是整个流程的完整体现，非常好 👍

```
请求 → 进入拦截器  
     ↳ 从 session 中获取 user  
        ↳ 有 → 保存到 ThreadLocal → 放行  
        ↳ 无 → 拦截返回 401  
请求结束 → 执行 afterCompletion → 清除 ThreadLocal  
```

---

## 🎯 小总结

| 概念        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| 拦截器      | 在请求到 Controller 前拦截请求，做统一处理（如登录校验）     |
| ThreadLocal | 在线程内共享数据（如当前登录用户），让后续代码能随时访问     |
| UserHolder  | 自定义封装的 ThreadLocal 工具类，用来保存/获取/清除当前用户信息 |

---

## 🔍 问题本质：你说的意思是——

> 我在 UserServiceImpl 中并没有写拦截器的调用代码，那 Spring 是怎么知道每次请求都要执行 LoginInterceptor.preHandle() 的？

---

## ✅ 真正起作用的是这段配置代码：

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
            .excludePathPatterns(
                "/voucher/**",
                "/upload/**",
                "/shop-type/**",
                "/shop/**",
                "/blog/hot",
                "/user/code",
                "/user/login"
            );
    }
}
```

### 📌 它做了两件大事：

1. **注册拦截器 LoginInterceptor 到 Spring MVC 的“拦截链”里**
2. 设置哪些路径要排除，哪些默认拦截

---

## 📦 Spring 是怎么做到拦截的？（内部流程）

你访问任何 URL，比如 /user/me 时：

1. **请求交给 DispatcherServlet（中央调度器）**
2. 它会去找：有哪些拦截器要拦？（你注册了拦截器）
3. Spring 执行你拦截器的 preHandle() 方法
4. 如果返回 true，就放行 → 进入 Controller
5. 如果返回 false，就结束请求（不再进入 Controller）

---

## 🎯 所以你不需要在 UserServiceImpl 或其他 Service 里手动调用拦截器

因为这是 Spring MVC 的核心功能，它会自动在 Controller 执行之前，**先走一遍你配置的所有拦截器链**

---

## ✅ 快速举例说明

你现在有这个接口：

```java
@GetMapping("/user/me")
public Result me(HttpSession session){
    UserDTO user = UserHolder.getUser();
    return Result.ok(user);
}
```

你没有在这段代码中加任何拦截或登录判断，因为：

- 登录校验早在 LoginInterceptor 的 preHandle() 中完成了
- 并且登录用户已经放进了 ThreadLocal
- 所以你只需要 UserHolder.getUser() 就能拿到用户信息！

---

## 🎁 一句话总结：

你之所以不需要在 UserServiceImpl 或 UserController 中主动调用拦截器，是因为你已经在 MvcConfig 中 **全局注册了拦截器**，Spring MVC 会在每次请求到达 Controller 之前，**自动执行拦截器链**。
