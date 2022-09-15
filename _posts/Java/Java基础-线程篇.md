---
title: Java基础-线程篇
abbrlink: ec1c4eca
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

