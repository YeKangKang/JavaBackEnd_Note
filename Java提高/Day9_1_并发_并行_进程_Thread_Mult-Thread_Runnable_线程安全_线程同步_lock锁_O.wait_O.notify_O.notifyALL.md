
# Day9 · Java 提高（并发、并行、进程、Thread、Mult-Thread、Runnable、线程安全、线程同步、lock锁、O.wait、O.notify、O.notifyALL）

本节系统梳理 Java 多线程模型、线程创建方式、线程调度机制、线程生命周期、同步机制（synchronized 与 Lock）、线程通信（wait/notify/notifyAll）等核心知识。

---

## 1. 并发与并行

| 概念                | 定义                               | 示例                        |
| ------------------- | ---------------------------------- | --------------------------- |
| 并发（Concurrence） | 多个任务在**同一时间段内交替执行** | 单核 CPU 上多个线程轮流运行 |
| 并行（Parallel）    | 多个任务在**同一时刻同时执行**     | 多核 CPU 上线程同时运行     |

> Java 多线程运行在操作系统层通常是“并发”，在多核 CPU 上才可能实现“并行”。

> “高并发”是从程序**设计/调度/资源管理角度**谈的挑战；
> “并行”是从**物理 CPU 核心层面**谈的执行能力；
> 真正的多线程程序多数都是 **并发为主**，**局部并行**。

---

## 2. Process 与 Thread

| 概念            | 定义说明                                           |
| --------------- | -------------------------------------------------- |
| 进程（Process） | 程序运行的基本单位，拥有独立内存空间               |
| 线程（Thread）  | 进程中的执行单元，共享进程内存资源，负责代码的执行 |

- 一个程序运行对应一个或多个进程；
- 一个进程中可包含多个线程，共享数据但调度独立。

---

## 3. 线程调度模型

### ✅ Java 使用“抢占式调度”：

- 高优先级线程会优先获得 CPU 使用权；
- 若多个线程优先级相同，则**随机选择一个**运行；
- 单核 CPU 上仍是并发执行，多核上才并行。

---

## 4. Thread 类线程创建方式

### ✅ 步骤：

1. 定义一个 `Thread` 子类，重写其 `run()` 方法；
2. 创建子类实例；
3. 调用 `start()` 方法启动线程（不能直接调用 `run()`）。

### ✅ 示例：

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("线程执行中：" + Thread.currentThread().getName());
    }
}

MyThread t1 = new MyThread();
t1.start(); // 正确启动线程
```

---

## 5. Runnable 接口线程创建方式（推荐）

### ✅ 特点：

- `Runnable` 是任务接口，不是线程类；
- 实现 `run()` 方法定义“线程要执行的任务”；
- 通过 `Thread(Runnable target)` 或 `Thread(Runnable target, String name)` 构造器包装成线程。

### ✅ 示例：

```java
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("任务正在运行：" + Thread.currentThread().getName());
    }
}

Runnable r = new MyRunnable();
Thread t = new Thread(r);
t.start();
```

---

## 6. Thread vs Runnable 对比

| 特性         | `Thread`（继承）         | `Runnable`（实现）          |
|--------------|---------------------------|-----------------------------|
| 是否线程类   | ✅ 是                     | ❌ 否，仅为任务类           |
| 资源共享     | ❌ 不方便                 | ✅ 支持多个线程共享任务对象 |
| 灵活性       | ❌ 已继承 Thread 无法再继承其他类 | ✅ 只实现接口，可多重继承 |
| 线程池支持   | ❌ 不支持直接提交         | ✅ 线程池推荐使用 Runnable  |

> 推荐使用 Runnable 接口方式创建线程，更灵活、资源共享友好、能兼容线程池调度。

---

---

## 7. 匿名内部类创建线程方式（快速写法）

### ✅ 三种匿名写法对比

#### 方式一：匿名实现 Runnable

```java
Runnable task = new Runnable() {
    public void run() {
        System.out.println("任务执行中");
    }
};
new Thread(task).start();
```

#### 方式二：匿名对象 + Thread 包装（推荐写法）

```java
new Thread(new Runnable() {
    public void run() {
        System.out.println("任务执行中");
    }
}).start();
```

#### 方式三：匿名继承 Thread 类

```java
new Thread() {
    public void run() {
        System.out.println("线程直接运行");
    }
}.start();
```

> 推荐使用方式二：结构清晰、可扩展；方式三虽可行但不推荐（不可复用、耦合高）

---

## 8. 多线程原理与 JVM 内存结构

- 启动程序时，JVM 创建主线程 `main()`，并为其分配独立线程栈空间；
- 每调用一次 `start()`，JVM 会创建一个新的线程栈并执行该线程的 `run()` 方法；
- 每个线程运行时的局部变量、方法调用都存在于各自线程栈中；
- 当所有线程执行完毕，该进程结束。

---

## 9. Thread 类构造器与常用方法

| 方法签名                            | 说明 |
|-------------------------------------|------|
| `Thread()`                          | 创建无名线程 |
| `Thread(String name)`               | 创建指定名称线程 |
| `Thread(Runnable target)`           | 用任务构建线程 |
| `Thread(Runnable target, String name)` | 用任务和名称构建线程 |
| `void start()`                      | 启动线程（创建栈 + 调用 run） |
| `void run()`                        | 线程体代码，不能手动调用替代 start |
| `static void sleep(long millis)`    | 当前线程睡眠指定毫秒 |
| `static Thread currentThread()`     | 获取当前执行线程的引用 |
| `String getName()`                  | 获取线程名称 |

> ⚠️ `sleep()` 不会释放锁，`wait()` 才会释放锁

---

## 10. 系统线程：main 与 GC（垃圾回收线程）

- Java 程序启动时，至少存在两个线程：
  - `main`：主线程，执行主逻辑
  - `GC`：JVM 垃圾回收线程，后台运行、自动触发
- 线程数可以通过工具如 `jconsole`、`jvisualvm` 观察

---

---

## 11. 线程状态（Thread.State 枚举）

| 状态             | 说明 |
|------------------|------|
| `NEW`            | 新建状态，线程对象已创建但未启动 |
| `RUNNABLE`       | 可运行状态，正在运行或等待 CPU 调度 |
| `BLOCKED`        | 被阻塞状态，等待获取对象锁 |
| `WAITING`        | 无限等待状态，需 `notify()` 唤醒 |
| `TIMED_WAITING`  | 限时等待状态，如 `sleep()`、`wait(timeout)` |
| `TERMINATED`     | 已终止，线程执行完毕或异常退出 |

---

## 12. 线程安全与竞争条件

当多个线程并发访问**同一个资源**，且至少一个线程对其进行了**修改操作**，就可能导致数据不一致，这种现象称为**线程安全问题 / 竞争条件（race condition）**。

---

## 13. synchronized 同步机制

### ✅ 同步代码块（推荐方式）

```java
synchronized (lockObject) {
    // 临界区代码，只能一个线程访问
}
```

- `lockObject` 可为任何引用类型，推荐使用 `private final Object lock = new Object();`
- 多个线程必须使用**同一把锁**，否则同步失效

---

### ✅ 同步方法

```java
public synchronized void doSomething() {
    ...
}
```

- 等价于 `synchronized (this) {}`；
- 若是 `static synchronized` 方法，则锁对象为 `类名.class`

> ⚠️ 使用 `this` 作为锁对象可能造成“同步策略暴露”，外部持有引用的代码也能 `synchronized(obj)` 参与锁竞争；

> 推荐使用私有锁对象，如 `private final Object lock = new Object();` 隔离同步控制逻辑。

---

## 14. Lock 接口与 ReentrantLock 类

### ✅ 核心特性

| 特性         | 说明 |
|--------------|------|
| 可重入       | 同一线程可重复获得同一锁，避免死锁 |
| 显式加锁/解锁 | 需手动调用 `lock()` / `unlock()` |
| 支持尝试锁     | `tryLock()` 可避免死等 |
| 可中断锁获取   | `lockInterruptibly()` |
| 可选公平性     | 构造时可传参是否公平调度 |

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // 临界区代码
} finally {
    lock.unlock(); // 必须释放锁
}
```

---

## 15. 线程通信机制（wait / notify / notifyAll）

- 定义在 `Object` 类中，必须在 `synchronized(obj)` 内使用；
- `wait()` 会让线程进入 `WAITING` 状态，并释放锁；
- `notify()` 唤醒**一个**等待线程（由JVM决定，不保证公平性）；
- `notifyAll()` 唤醒**所有**等待线程；
- 唤醒后线程进入 `RUNNABLE`，但还需重新竞争锁才能真正运行；
- `wait(long timeout)` 进入 `TIMED_WAITING` 状态，超时自动苏醒。

---

## 16. sleep vs wait 比较

| 对比项         | `sleep()`           | `wait()`                   |
|----------------|---------------------|----------------------------|
| 所属类         | `Thread`            | `Object`                   |
| 是否释放锁     | ❌ 否                | ✅ 会释放                  |
| 是否需同步块包裹 | ❌ 否                | ✅ 是                      |
| 是否可自动唤醒 | ✅ 时间到自动唤醒    | ✅ 时间到或被唤醒均可      |
| 通常用途       | 单线程延迟执行       | 多线程间通信               |

---

## ✅ 总结

- Java 多线程模型核心包括线程创建（Thread、Runnable）、线程调度与状态、资源同步（synchronized、Lock）及通信机制（wait/notify）；
- 推荐使用 `Runnable` + 线程池管理任务，避免继承 Thread；
- 线程同步时尽量使用私有锁对象，确保封装性；
- 可重入锁（ReentrantLock）适合复杂同步需求；
- `wait()` / `notify()` 是对象锁级别通信方式，需在同步块中使用；
- `sleep()` 是线程行为，与锁机制无关。

