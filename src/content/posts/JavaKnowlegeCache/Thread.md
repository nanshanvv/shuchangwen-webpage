---
title: How to create Thread in java
published: 2025-04-04
tags: [Java]
category: JavaKnowledgeCache
description: Guide of creating Thread in java
draft: false
---

## 💡 Java创建线程的4种主要方式

### ✅ 方法 1：继承 Thread 类（最基础）

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Hello from thread!");
    }
}

public class Main {
    public static void main(String[] args) {
        Thread t = new MyThread();
        t.start(); // 启动线程（不能用 run()）
    }
}
```

**✅ 简单易懂，但不推荐用于实际开发：**

- Java 只能单继承，如果你继承了 `Thread` 就不能再继承其他类了。
- **不能直接调用 `run()` 的原因：** `run()` 方法只是一个普通方法，直接调用它不会开启新线程，而是同步执行代码（在主线程中执行），必须用 `start()` 才能创建并启动一个新的线程。

------

### ✅ 方法 2：实现 Runnable 接口（更灵活）

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable thread is running!");
    }
}

public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start();
    }
}
```

**✅ 推荐！** 可以避免继承限制，适合多任务处理、复用逻辑。

------

### ✅ 方法 3：使用 Lambda 表达式（更简洁）

适用于 Java 8+，配合 Runnable：

```java
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            System.out.println("Thread with lambda!");
        });
        t.start();
    }
}
```

**✅ 简洁优雅，开发中常用。**

------

### ✅ 方法 4：使用线程池（Executor 框架）🌟推荐！

线程池能复用线程，提升性能，避免频繁创建销毁线程。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        executor.submit(() -> {
            System.out.println("Thread in pool is running!");
        });

        executor.shutdown(); // 关闭线程池（不再接受新任务）
    }
}
```

------

## 🧠 补充知识点：线程池常见类型

| 方法                                  | 说明                                   |
| ------------------------------------- | -------------------------------------- |
| `Executors.newFixedThreadPool(n)`     | 固定大小的线程池（适合并发固定任务）   |
| `Executors.newCachedThreadPool()`     | 可自动扩容的线程池（适合任务数不确定） |
| `Executors.newSingleThreadExecutor()` | 单线程线程池（适合顺序执行任务）       |
| `Executors.newScheduledThreadPool(n)` | 可定时/周期性执行任务的线程池          |

------

## ❗警告：不要直接使用 `new Thread()` 多次启动任务！

每次创建新线程会增加系统负担，无法统一管理线程资源；使用线程池可以避免这些问题。

------

## 🛠 实战：线程池 + 锁 + 异常处理（结合缓存重建代码）

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

if (tryLock(lockKey)) {
    executor.submit(() -> {
        try {
            saveShop2Redis(id, 30L);  // 更新缓存
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            unLock(lockKey); // 必须释放锁
        }
    });
}
```
