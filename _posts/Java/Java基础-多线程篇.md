---
title: Java基础-多线程篇
date: 2022-08-15 05:24:51
categories:
  - Java
  - Thread
tags: [Java, Thread]
---

> 介绍多线程的使用与案例

<!-- more -->



# 线程池
为每个任务单独从池子中获取线程, 具有高效和快速..
避免了系统层面创建大量线程的开销问题

## 好处
1. 加快响应速度
2. 合理利用CPU和内存
3. 方便管理

## 适用场景
1. 服务器线程池
2. 需要多个线程处理任务

## 使用
## 创建线程池
### 构造参数
| 参数            | 类型                       | 含义                                                         |
| --------------- | -------------------------- | ------------------------------------------------------------ |
| `corePoolSize`  | `int`                      | 核心线程数, 在线程池初始化完成后,默认没有线程,等任务到来后,再去创建线程,常驻的线程数量 |
| `maxPoolSize`   | `int`                      | 最大线程数, 在核心线程数基础上, 额外添加一些线程的数量上线   |
| `keepAliveTime` | `long`                     | 保持存活时间, 在线程多余核心线程数量且线程空闲时间超过存活时间, 则回收空闲线程 |
| `workQueue`     | `BlockingQueue`            | 任务存储队列                                                 |
| `threadFactory` | `ThreadFactory`            | 当线程池需要新的线程的时候, 会使用 `ThreadFactory` 来生成新的线程. 默认使用 `Executors.defaultThreadFactory()` 创建, 具有相同线程组和优先级 |
| `Handler`       | `RejectedExecutionHandler` | 由于线程池无法接受你所提交的任务的拒绝策略                   |

### 添加规则
1. 如果线程数小于 `corePoolSize`，即使其他工作线程处于空闲状态，也会创建一个新线程来运行新任务。
2. 如果线程数等于(或大于)  `corePoolSize` 但少于 `maximumPoolSize`，则将任务放入队列。
3. 如果队列数已满，并且线程数小于`maxPoolSize`，则创建一个新线程来运行任务。
4. 如果队列数已满，并且线程数大于或等于`maxPoolSize` ,则拒绝该任务。

#### 特点
1. 通过设置 `corePoolSize` 和 `maximumPoolSize` 相同，就可以创建固定大小的线程池。
2. 线程池希望保持较少的线程数，并且只有在负载变得很大时才增加它。
3. 通过设置 `maximumPoolSize` 为很高的值，例如 `Integer.MAX_VALUE`，可以允许线程池容纳任意数量的并发任务。
4. 只有在队列填满时才创建多于 `corePoolSize` 的线程，所以如果你使用的是无界队列（例如 `LinkedBlockingQueue` )，那么线程数就不会超过 `corePoolSize`。

#### 工作队列
`SynchronousQueue`
直接交接队列, 没有容量, 在收到任务后直接交给线程执行,不会在队列停留

`LinkedBlockingQueue`
无界队列, 容量无限, 将所有的任务都放到队列中..如果处理的慢,可能会内存溢出

`ArrayBlockingQueue`
有界队列, 可以设置队列大小.

`DelayedWorkQueue`
延迟队列, 可以延迟工作

### 自动常用线程池构建
`newSingleThreadExecutor`
核心线程数量与最先线程数量均为1, 无存活时间, 队列使用 `LinkedBlockingQueue` 数量无限, 当队列积压过多时, 容易内存溢出.

`newFixedThreadPool`
线程固定数量,核心线程数与最大线程数一致, 无存活时间, 队列使用 `LinkedBlockingQueue` 数量无限, 当队列积压过多时, 容易内存溢出.

`newCachedThreadPool`
核心线程数为0, 最大线程数为 `Integer.MAX_VALUE`, 默认存活时间60S, 队列使用 `SynchronousQueue` 直接将任务交给线程, 当任务到来时, 直接创建新的线程完成任务, 超时自动回收多余线程.. 可能为会创建非常多的线程

`newScheduledThreadPool`
设置核心线程后, 最大线程数为 `Integer.MAX_VALUE`, 无存活时间, 具有延时或者周期性任务的线程池.. 

### 手动创建建议
CPU 密集型任务(加密, 计算), 最佳线程数为 CPU 核心线程数的 1-2 倍
耗时IO任务(数据库, 文件, 服务请求), 线程可以多点, 以 JVM 线程监控显示为依据.

线程数 = CPU 核心数 X (1 + 平均等待时间/平均工作时间)

### 停止线程池

| 命令               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| `shutdown`         | 初始化关闭,等线程和队列中的任务完成后再关闭                  |
| `isShutdown`       | 判断当前线程池是否在停止状态                                 |
| `isTerminated`     | 线程是否完全关闭                                             |
| `awaitTermination` | 等待一段时间, 如果当前10秒后还未关闭, 返回 False             |
| `shutdownNow`      | 立刻关闭线程池. 并将未执行的线程任务返回.. 如果线程被中断了, 抛出 `java.lang.InterruptedException` 异常 |

### 线程拒绝策略
`AbortPolicy`   拒绝任务时, 抛出异常
`DiscardPolicy` 拒绝任务, 且没有任何提示
`DiscardOldestPolicy` 丢掉最老的任务, 腾出空间给新任务
`CallerRunsPolicy` 将多余的线程,向上抛给提交线程去执行 

### 钩子函数
通过锁, 对线程进行等待和唤醒

### 线程池组成
- 线程池管理器  管理线程的类
- 工作线程      生成的任务线程对象
- 任务队列    队列的类
- 任务接口    具体的任务类

### 常见类关系
`Executor` 
顶级接口,定义 `execute` 执行方法

`ExecutorService`
继承自 `Executor` 接口, 并扩展提供 `shutdown`, `shutdownNow` 等线程管理方法

`ThreadPoolExecutor`
线程池, 具体的实现类, 实现上述接口

`Executors`
工具类,快速创建线程池

### 线程池状态

| 状态         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| `RUNNING`    | 接受新任务并处理排队任务                                     |
| `SHUTDOWN`   | 不接受新任务, 但处理排队任务                                 |
| `STOP`       | 不接受新任务, 也不处理排队任务, 并中断正在进行的任务         |
| `TIDYING`    | 中文是整洁, 理解了中文就容易理解这个状态了: 所有任务都已终止, `workerCount` 为零时, 线程会转换到 `TIDYING` 状态, 并将运行`terminate()` 钩子方法。 |
| `TERMINATED` | `terminate()` 方法运行完成                                   |



# `ThreadLocal` 介绍

## 使用场景
1. 每个线程需要一个独享的对象,通常是一个静态工具类..如 `Random` 类, `SimpleDateFormat` 类
2. 每个线程需要保存全局变量(如拦截器中获取用户信息), 让不同方法中直接使用, 避免参数传递的麻烦

### 每个线程需要独立副本
通过将静态成员变量, 放入到 `ThreadLocal` 中进行包装. 在每个线程使用时, 获取线程对应的静态成员变量, 避免多个线程间共用同一个对象

### 每个线程保存全局变量
将当前线程中共有的变量, 通过 `ThreadLocal` 中通过 `set/get` 进行存放

## 原理类

| 接口类           | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| `Thread`         | 线程, 每个线程中有成员变量 `threadLocals`, 存放一个 `Map`, 为 `ThreadLocalMap` 对象 |
| `ThreadLocalMap` | `Map` 结构, 持有 `ThreadLocal` 对象, 因为一个线程中, 可能关联有多个 `ThreadLocal` |
| `ThreadLocal`    | 具体的线程对应的变量相绑定                                   |

## 常用方法
- **`T setInitialValue()`**

1. 返回当前线程的初始值. 懒加载, 只有在 `get` 时才会执行
2. 当线程第一次使用 `get` 方法时, 将调用; 如果在此之前, 先调用了 `set` 方法, 则该 `setInitialValue` 方法不会被执行
3. 通常, 每个线程最多调用一次此方法, 但是如果已经调用了 `remove` 后, 再次 `get` 时, 会再次调用此方法
4. 如果不重写此方法, 则方法会返回 `null` , 一般我们会在创建阶段完成初始化.

- **`void set(T)`**
  为线程设置一个新值

- **`T get()`**
  得到线程对应的 `value` , 如果首次调用, 会执行 `initialValue` 方法
  `get` 方法是先取出当前线程的 `ThreadLocalMap`, 然后调用 `map.getEntry` 方法, 把本 `ThreadLocal` 的引用作为参数传入，取出 `map` 中属于本 `ThreadLocal` 的 `value`
  **注意，这个 `map` 以及 `map` 中的 `key` 和 `value` 都是保存在线程中的，而不是保存在 `ThreadLocal` 中**

- **`void remove()`**
  删除当前线程对应的值

## Value 内存泄露问题
`ThreadLocalMap` 的每个 `Entry` 都是一个对 `key` 的弱引用; 同时, 每个 `Entry` 都包含了一个对 `value` 的强引用
正常情况下, 当线程终止, 保存在 `ThreadLocal` 里的 `value` 会被垃圾回收, 因为没有任何强引用了
但是, 如果线程不终止(比如线程需要保持很久), 那么 `key` 对应的 `value` 就不能被回收, 因为有以下的调用链:
`Thread —> ThreadLocalMap —> Entry(key为null) -> Value`

因为 `value` 和 `Thread` 之间还存在这个强引用链路, 所以导致 `value` 无法回收, 就可能会出现 OOM
JDK已经考虑到了这个问题, 所以在 `set`, `remove`, `rehash` 方法中会扫描 `key` 为 `null` 的 `Entry`, 并把对应的 `value` 设置为 `null`, 这样 `value` 对象就可以被回收;
但是如果一个 `ThreadLocal` 不被使用, 那么实际上 `set`, `remove`, `rehash` 方法也不会被调用. 如果同时线程又不停止, 那么调用链就一直存在, 导致了value的内存泄漏

我们主动调用 `remove` 方法, 避免内存泄露

## 常用类
`RequestContextHolder`  Spring中存放http请求与响应的工具类



# 锁
## Lock 接口
锁是一种工具, 用于控制共享资源的访问 常用如 `ReentrantLock` 

### synchronized 

1. 效率低, 释放情况少(1.代码完全执行完成;2.抛出异常)
2. 获得锁的动作不能设定超时
3. 无法知道是否成功获得锁, 在尚未获取锁之前会一直等待...

### Lock 接口

`lock`                      获取锁, 如果当前锁已经被其他线程获取, 则等待; Lock 不会像 synchronized 那样在发生异常时解锁, 因此一定要在 finally 中释放
`tryLock`                   尝试获取锁, 可以设置超时时间, 如果依然没有拿到锁, 返回 false!
`lockInterruptibly`         相当于在尝试获取锁时将超时时间设置为无线,在等待锁的过程中, 可以被打断,抛出 InterruptedException 异常
`unlock`                    解锁, 一定要写在 finally 语句中, 千万不要忘记释放锁

## 标准用法
标准写法,在锁后必须跟上try执行业务逻辑和finally释放锁
```java
// 上锁
lock.lock();
// try 需要被独占的资源
try {
    System.out.println("业务逻辑.....");
} finally {
    // 必须释放锁 
    lock.unlock();
}
```

## 可见性
线程与线程之间数据的状态,保证不同线程之间可以看到对方的数据发生修改

> happens-before 原则: 我们这件事发生了, 别的线程能看到发生的事件, 知晓修改的内容;

Lock 和 synchronized 具有相同语义, 可以保证下一个线程在加锁后可以看到所有前一个线程解锁前发生的所有操作. 


## 锁的分类
| 说明                               | 锁                                              | 示例       |
| ---------------------------------- | ----------------------------------------------- | ---------- |
| 线程要不要锁住同步资源             | 锁住/悲观锁                                     | for update |
| 多线程能否共享一把锁               | 共享锁/独占锁                                   | 读写锁     |
| 多线程竞争时，是否排队             | 排队公平锁/非公平锁(先尝试插队，插队失败再排队) |            |
| 同一个线程是否可以重复获取同一把锁 | 可以可重入/不可重入锁                           |            |
| 可以可中断锁                       | 可中断锁/非可中断锁                             |            |
| 等锁的过程                         | 自旋锁(自旋,尝试获取锁)/非自旋锁(阻塞)          |            |

### 乐观锁与悲观锁
非互斥同步锁/互斥同步锁