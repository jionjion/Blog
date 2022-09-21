---
title: Java基础-线程篇
abbrlink: ec1c4eca
typora-root-url: ../../
date: 2022-08-15 05:24:51
categories:
  - Java
  - Thread
tags: [Java, Thread]
---

> 介绍Java中线程的使用

<!-- more -->



# 线程篇

## 基础知识

### 创建线程

- 继承 `Thread` 类, 重写 `run` 方法

- 实现 `Runnable` 接口, 实现 `run` 方法, 在构建 `Thread` 对象时, 传入 `Runnable` 实现类

 

#### 源码解析

在 `Thread` 类中的 `run` 方法, 通过判断是否在构造函数中传入 `Runnable` 接口, 来确定是否执行 `Runnable` 接口..

```java
public class Thread implements Runnable {
    // Runnable 接口实现类
    private Runnable target;    
    
    // 构造函数, 传入 Runnable 接口实现类
    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }    
    
    @Override
    public void run() {
        // 首先判断是否传入 Runnable 接口
        if (target != null) {
            // 传入了, 则执行 Runnable 接口实现类中的 run 方法 
            target.run();
        }
    }
}
```





### 启动线程

`start` 方法

通知虚拟机在空闲下, 请求启动新线程... (注意, 只是请求启动, 具体的调用由线程调度器执行, 可能不会立即执行, 比如线程饥饿)..
所以, 不同线程的在程序中的调用顺序并不能决定具体线程的启动顺序..



### 子线程准备

 `start` 方法是由主线程启动, 启动后会有个新的子线程..
子线程在创建后会进入 **就绪** 状态, 表示其已经获得了除CPU以外的资源, 比如 线程上下文, 程序栈, 程序计数器..

随后进入 **执行** 状态, 等待CPU资源, 最后再进入 **运行** 状态



#### 源码解析

所以,  `start()` 方法不能重复执行, `start0()` 方法为一个 `private native void start0();` 方法, 不能够直接看到...

```java
public class Thread implements Runnable {
    public synchronized void start() {
        // 检查线程状态
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        // 加入线程组
        group.add(this);

        // 调用 start0 启动
        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                      it will be passed up the call stack */
            }
        }
    }
}
```



### 停止线程

使用 `interrupt` 通知线程, 来进行停止线程... 而不是强制, 即线程的中断不是被强行停止的, 而是相互通知协商停止..



#### 线程停止场景

1. `run` 方法正常结束
2. 线程抛出异常, 被捕获后释放线程..



#### 错误停止线程

`stop` 方法, 已被弃用, 立即结束程序, 破坏原有的原子操作..
`suspend` 和 `resume` 方法, 挂起线程, 此时线程是可以带锁的..如果需要同样该锁的线程去唤醒,则会造成死锁...



#### 正常停止线程 - `interrupt` 

通过使用 `interrupt()` 方法, 并且在函数中通过 `isInterrupted()` 方法判断是否线程停止, 来正确停止线程..



常用方法..

| 方法                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `static boolean interrupted()` | 是否被中断, 如果返回 `true` 后表示线程被中断, 同时 **清除中断标志...** |
| `boolean isInterrupted()`      | 是否被中断, 返回 `true` 当前线程已经被中断了**...不清除中断标志...** |
| `Thread.interrupted()`         | 对**当前线程**进行中断, 同时**打上中断标志...**              |



#### 停止阻塞的线程

默写方法, 如 `Thread.sleep()` 方法, 会在方法中抛出 `InterruptedException` 异常, 由此来执行收到 `interrupt` 信号后的停止线程任务..



### 正确停止线程实践

#### 传递中断

如果在业务方法中抛出跟线程相关的异常, 如 `InterruptedException` 异常, 我们应该继续在方法声明中主动向上抛出... 而非自行处理..

```java
/**
  *  业务方法, 在业务方法中主动声明, 线程被打断后的业务逻辑... 
  */
private void throwInMethod() throws InterruptedException{
    // 休眠10秒, 方法会抛出 InterruptedException 异常. 业务方法中不作处理, 向上抛出..
    Thread.sleep(10000);
}
```



#### 恢复中断

如果不想传递异常或者无法传递异常时, 需要调用 `Thread.currentThread().interrupt();` 告知线程当前被中断...

```java
/**
  * 业务方法, 在业务方法中捕获异常, 并将当前线程标记为被中断
  */
private void reInterrupt() {
    // 休眠10秒
    try {
        Thread.sleep(10000);
    } catch (InterruptedException e) {
        // 恢复设置当前异常为中断状态
        Thread.currentThread().interrupt();
        e.printStackTrace();
    }
}
```



#### 响应中断的方法汇总

这些方法在执行时, 线程会被阻塞, 因此会抛出 `InterruptedException` 异常..

| 方法                                                | 说明                                          |
| --------------------------------------------------- | --------------------------------------------- |
| Object.wait()                                       | 对象的等待操作                                |
| Thread.sleep()                                      | 线程休眠                                      |
| Thread.join()                                       | 当前线程阻塞, 加入到父线程组中                |
| java.util. concurrent.BlockingQueue.take() / put()  | 队列的获取和放下, 阻塞                        |
| java.util.concurrent.locks.Lock.lockInterruptibly() | 在加锁期间, 线程阻塞                          |
| java.util.concurrent.CountDownLatch.await()         | 倒数门栓的等待                                |
| java.util.concurrent.CyclicBarrier.await()          | 循环计数器的等待                              |
| java.util.concurrent.Exchanger.exchange()           | 交换队列的交换动作.如果没有新的元素入队, 等待 |
| java.nio.channels.InterruptibleChannel 类           | NIO 相关操作                                  |
| java.nio.channels.Selector 类                       | NIO 相关操作                                  |



### 线程的状态

- `new` 新建, 线程新建, 但是还未执行, `start()` 方法, 此时线程已经做了一些准备, 但尚未运行 `run()` 方法
- `runnable` 可运行, 掉用 `start()` 方法后, 处于可运行(等待CPU分配时间)或者运行中的状态(CPU正在执行)
- `blocked` 阻塞, 线程有且只有进入 `synchronized` 修饰的代码块, 且未获取锁的时候, 进入排队状态
- `waiting` 等待, 线程进入锁, 计时器, `Object.wait` , `Thread.join()` 时, 处于等待状态, 直到锁释放, 计时结束, `Object.notify()/notifyAll()` 结束等待
- `timed waiting` 计数等待, 线程进入可中断锁锁, 休眠 `Object.wait(time)` , `Thread.join(time)` 时, 处于计时等待状态, 直到锁释放或超时,  `Object.notify()/notifyAll()` 结束计时等待
- `terminated` 终止, 无论是否正确结束或抛出异常, 线程执行完毕.



<img src="/images/2022-09/线程状态转换.png" alt="线程状态转换" style="zoom:67%;" />

特殊情况:

- 当从`Object.wait()` 状态唤醒时, 通常不会理解抢到 `monitor` 锁, 那么线程状态会从 `waiting` 先进入到 `blocked` 状态, 抢到锁后再转换到 `runnable` 状态
- 如果发生异常, 可以直接到 `terminated` 状态, 不再遵循路径规定



## Thread 类

| 方法                            | 简介                   |
| ------------------------------- | ---------------------- |
| `Object.wait/notify/ notifyAll` | 让线程暂时休息和唤醒   |
|                                 |                        |
| `sleep`                         | 休眠线程               |
| `join`                          | 等待其他线程执行完毕   |
| `yield`                         | 放弃已经获取的CPU资源  |
| `currentThread`                 | 获取当前执行线程的引用 |
| `start`, `run`                  | 启动线程               |
| `interrtupt`                    | 通知中断线程           |
| ~~`stop`. `suspend`, `resume`~~ | ~~挂起线程, 启用~~     |


因为 `wait` , `notify` , `notifyAll`  需要一个监视器对象 `monitor` , 将锁的信息保存在对象或者类上, 因此将其放到 `Object` 对象类中合适
而  `sleep` 等线程方法, 因为线程在执行过程中可能获得多把锁, 所以放到 `Thread` 类中.



### 线程阻塞/唤醒

`wait` , `notify` , `notifyAll` 对线程的阻塞与唤醒

#### 阻塞阶段

线程处于阻塞阶段时, 不再被 CPU 执行, 直到被唤醒..

- 本线程调用锁的 `notify()` 方法
- 其他线程调用这个锁对象的 `notifyAll()` 方法
- 超过了 `awit` 设置的超时时间
- 线程自身调用 `interrupt()` 方法

#### 唤醒阶段

在 ``synchronized` 代码块中, 执行了 `notify` 和 `notifyAll` 方法, 将当前锁对象的 `monitor` 释放, 线程则会被唤醒, 可以被CPU继续执行..

#### 遇到中断

在线程执行, 调用了 `interrtupt` 方法, 线程被中断

#### 注意点

- 在阻塞/唤醒前必须拥有 `monitor`  资源监视器, 即 `wait` , `notify` , `notifyAll` 方法必须在 `synchronized (monitor)` 同步代码块中执行, 同时要求对应的资源监视器必须一样..
- `notify` 只会唤醒一个线程.. 多个线程下无法获得..
-  `wait` , `notify` , `notifyAll`  属于 `Object` 类的 `public final native void` 方法, 不能被修改和看到
- 类似于 `Codition`类 , 条件解锁.
- 如果持有多个锁的情况, 注意释放锁的顺序..



#### 内部原理

- 默认在资源监视器 `monitor` 中, 有多两个集合, 入口集 和 等待集.
- 线程在进入入口集后, 会作争抢, 只有一个线程被接受 `acquire`, 放入到执行中, 在执行中有两种情况..
  1. 当执行完毕后, 会释放线程, 结束线程流程..
  2. 遇到 `wait` 方法后, 会阻塞当前线程, 将当前线程放入到等待集中.

- 当有 `notify` 或者 `notifyAll` 方法触发当前监视器 `monitor` 后, 从等待集中争抢, 获得一个线程进行执行...

<img src="/images/2022-09/wait原理.png" alt="wait原理" style="zoom:50%;" />



### 线程类方法

#### `sleep` 方法

1. 暂时休眠, 进入到等待或者计时等状态,  但是在休眠期间, **不释放锁** `Lock` 类及其实现  和 `synchronized` 同步代码块的锁
2. 如果休眠期间被打断后抛出 `InterruptedException` 异常, 并且清除中断状态

#### `join` 方法

1. 新的线程加入, 等待新的子线程执行完成后, 主线程再继续执行
2. 主线程被阻塞时..

##### Join 源码

在 `join` 方法, 内部调用了 `wait` 方法, 将当前线程进行等待. 此时主线程线程状态为 `WAITING` 等待状态, 子线程继续执行
另外, 在每一个 `Thread` 类执行完成后, 都会调用 `notifyAll` 方法. 因此, 子线程结束后, 会通知因为它被阻塞的主线程, 从而将主线程的状态从 `WAITING` 等待的状态转为运行 `RUNNABLE` .

```java
public final synchronized void join(long millis) throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            // 调用 wait 方法, 使线程进入无限期等待..
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}

// 等待方法
public final native void wait(long timeout) throws InterruptedException;
```



##### 推荐

可以通过 `CountDownLatch` 或者 `CyclicBarrier` 类进行替换...达到流程控制的作用



#### yield 方法

释放当前线程的 CPU 时间片且不放弃锁, 但是 jvm 并不一定遵守... 
如果当前CPU不忙, 调用 `yield` 方法不一定释放CPU资源...



### 线程属性

 属性概览

| 属性     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| id       | 线程唯一编号, 主线程为1, 后续新建线程自增..不允许被修改      |
| name     | 线程名, 默认通过 `Thread + 序号` 构成, 或者在创建线程时通过构造函数指定名称<br />也可以通过 `setName()` 方法主动设置, 但是启动后只能修改 `java` 层面的线程明, 操作系统层面线程名称启动后不可变 |
| isDaemon | 是否为守护线程(`true`) 与 用户线程 (`false`) ;               |
| priority | 线程优先级, 告诉线程调度器                                   |



#### 守护线程

为用户线程提供基础服务, 随虚拟机启动/关闭.

- 线程类型默认继承父线程, 守护线程创建的线程默认也是守护线程
- 守护线程被 `jvm` 启动, (除了 `main` 线程, 被 `jvm` 启动, 但性质为用户线程)
- 不影响 `jvm` 的退出, 当没有用户线程后, `jvm` 就会退出



#### 优先级

`JVM` 优先级共 10 个级别 (1~10 个级别, 10最高), 默认为 5.. 剩余为`1`, `10` ..一般不做指定

程序的设计不应该设计依赖于优先级, 不同操作系统对优先级理解不同..(linux 就没有优先级概念..)
另外, 优先级也会被操作系统修改(windows 会将多次请求执行的线程优先级主动提高..)



### 异常处理

#### 问题

- 子线程中抛出异常, 并不会影响主线程运行
- 同时抛出异常也不明显..日志容易被忽略
- 无法通过 `try-catch` 捕获, 子线程异常无法用传统方法捕获

#### 解决

- 子线程的问题交给子线程自身通过 `try-catch` 进行捕获处理
- 通过 `UncaughtExceptionHandler` 进行的全局异常处理.. 默认有 `java.lang.ThreadGroup` 异常处理器处理..

```java
// 默认异常处理
public void uncaughtException(Thread t, Throwable e) {
    // 首先调用父类的异常处理, 一般为空
    if (parent != null) {
        parent.uncaughtException(t, e);
    } else {
        // 调用 Thread.setDefaultUncaughtExceptionHandler() 中设置的默认处理, 进行全局异常处理
        Thread.UncaughtExceptionHandler ueh =
            Thread.getDefaultUncaughtExceptionHandler();
        if (ueh != null) {
            ueh.uncaughtException(t, e);
        } else if (!(e instanceof ThreadDeath)) {
            // 如果全局异常处理还不存在, 就输出异常..
            System.err.print("Exception in thread \"" + t.getName() + "\" ");
            e.printStackTrace(System.err);
        }
    }
}
```



 ### 全局异常处理

- 程序全局异常处理
- 每个线程单独设置
- 线程池统一设置

自定义实现异常处理类

```java
// 自定义
public class MyUncaughtExceptionHandler implements Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread thread, Throwable throwable) {
        // 处理异常业务..
        throwable.printStackTrace();
        System.out.println(thread.getName() + "异常发生了, " + throwable.getMessage());
    }

    // 测试使用, 在创建线程时, 设置默认的异常处理...
    public static void main(String[] args) {
        // 在线程执行中, 抛出异常..
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    throw new RuntimeException(Thread.currentThread().getName() + "抛出了异常");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        // 设置全局的异常处理器
        Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandler());

        // 调用会抛出异常的线程
        new Thread(runnable, "线程-1").start();
    }
}

```

#### 每个线程单独设置

通过 `Thread.setUncaughtExceptionHandler` 方法单独设置..

```java
public static void main(String[] args) {
    Thread thread2 = new Thread(runnable, "线程-2");
    // 每个线程单独设置
    thread2.setUncaughtExceptionHandler(new MyUncaughtExceptionHandler());
    thread2.start();
}
```



## 线程问题

### 线程安全

当读个线程访问一个对象时, 如果不用考虑线程运行环境下的调度和交替执行, 也不用进行额外的同步, 或者在调用方进行其他的协调操作, 调用这个对象行为都可以获得正确的结果, 那么这个对象就是线程安全的..

#### 常见线程安全问题

- 线程争抢相同资源问题

​		比如在多个线程的 `run` 方法中操作同一个对象变量, 静态变量, 共享缓存, 数据库

- 线程调用时序问题

  比如各种数据库读后修改写入 `read-modify-write` 问题, 程序检查后再运行 `check-then-act` 问题

- 不同数据之间存在绑定关系

- 对象发布和初始化时候的问题, 造成对象溢出(本意未完成创建的对象, 但是由于代码问题, 就可以被外部访问) ..

  可以通过返回副本(深拷贝原对象), 或者工厂模式完成安全发布..

  

### 性能损耗

#### 上下文切换

当可运行的线程数量,超过计算机CPU核心数后能够支持的线程数量时, 会进行线程间切换.
调度上下文切换时, 会挂起线程, 保存线程状态, 对程序造成性能影响..
容易造成缓存占用过多, 缓存失效等问题..

密集的上下文切换多发生在争抢锁, 数据库操作, 各种 `IO` 过程.

#### 内存同步

指令重排序, 自动加锁/释放锁, `volatile` 关键字同步变量
