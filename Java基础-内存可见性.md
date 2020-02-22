---
title: Java基础-内存可见性
abbrlink: 814a6883
date: 2020-01-28 10:22:08
categories:
  - Java
  - JVM
tags: [Java, JVM]
---



# 内存可见性介绍

## 简介

该项目围绕内存模型的变量可见性进行展开,对多线程编程中常见的内存争议问题进行复现和解决.




## 示例代码

* *[SynchronizedDemo][1]*                   使用synchronized关键字,修饰实现内存可见
* *[VolatileDemo][2]*                             使用volatile,修饰实现内存可见



## 内存可见性介绍

### 名词解释

**可见性:** 一个线程对共享变量值的修改,能够及时地被其他线程看到,则该变量为可见的
**共享变量:** 如果一个变量在多个线程的工作内存中都存在副本,那么这个变量就是这几个线程的共享变量
**Java内存模型:** 描述了java程序中各种共享变量(线程共享变量)的访问规则,以及在JVM中将共享变量储存到内存和从内存中读取出共享变量这样的底层细节
**主内存:** 所有的变量都储存在主内存中
**工作内存:** 每个线程都有自己的独立的工作内存,里面保存该线程使用的变量副本(主内存中该变量的一份拷贝)

### 工作规则

 1. 线程对共享变量的所有操作都必须在自己的工作内存中进行,不能直接从主内存中进行读写
 2. 不同线程之间无法直接访问其他线程工作内存中的变量,线程间变量值的传递需要通过主内存完成

### 工作方式
首先把工作内存中的值刷新到主内存中,然后再把主内存中变更的值刷新到工作内存2中,实现不同线程间的同一个变量的调用.

### 注意

 1. 线程修改后的共享变量值能够及时从工作内存刷新到主内存中
 2. 其他线程能够及时把共享变量的最新值从主内存中刷新到自己的工作内存中

## 内存可见性的实现方式
### `synchronized` 关键字实现

同步,实现步骤锁,保证同一时刻只能有一个修改
线程解锁前,必须把共享变量的最新值刷新到主内存中
线程加锁前,将清空工作内存中共享变量的值,从而使用共享变量时需要从主内存中重新读取最新的值
**工作顺序:** 获得互斥锁->清空工作内存->主内存拷贝最新副本到工作内存->执行代码->将更改后的共享变量刷新到主内存->释放互斥锁

### `volatile` 关键字实现

可以保证变量的可见性,但是不能保证复合操作的原子性
加入内存屏障和进制重排序.
对于写操作,会在操作后加入一条`store`屏障指令,刷新主内存
对于读操作,会在操作后加入一条`load`屏障指令,刷新工作内存
volatile变量在每次被线程访问时,总会从主内存中重读最新值;当改变量发生改变是,又会刷新到主内存中.保证任何时刻不同线程总能看到该变量的最新值
**适用情况**

 1. 对变量的写入作不依赖当前值,改变后的值不能与原来的值有关系.如:各种变化,boolean变化
 2. 该变量没有包含在其他变量的不变式中.	如:两个变量之间的大小比较

### 比较

 1. `volatile`不需要加锁,比`synchronized`更轻量级,不需要阻塞线程
 2. `volatile`的读相当于加锁,`volatile`的写相当于解锁
 3. `synchronized`既可以保证可见性,又可以保证原子性.而`volatile`不能保证原子性

### 重排序

代码书写顺序与实际执行顺序不同,指令重排序是为了提高程序性能而做出的优化

 - 编译器优化的重排序(编译器优化)
 - 指令级并行重排序(处理器优化)
 - 内存系统的重排序(处理器优化)

`as-if-serial`:无论如何重排序,程序执行的结果应该与代码顺序执行的结果一直(单线程下遵循如此)

### 共享变量在线程间不安全的原因:
1. 线程的交叉执行
2. 重排序结合线程交叉执行
3. 共享变量更新后的值没有在工作内存与主内存间及时更新

# 代码实现
## 需求描述
在类中定义了一组共享变量,它们可以被一对`write()`方法和`read()`进行修改和读取.通过继承`Thread`类重写`run()`方法,进行多线程执行.我们在`main()`方法中**多次调用,可以得到不同不的结果.**

 - 修改前代码

``` java
public class SynchronizedDemo {

	/**共享变量*/
	private boolean ready = false;	//是否可读标志
	private int result = 0;
	private int number = 1;
	/**写操作,为共享变量写入最新值*/
	public  void write() {
		ready = true;				//①
		number = 2;					//②
	}
	/**读操作,为共享变量读取最新值*/
	public  void read() {
		if (ready) {				//③
			result = number * 3;	//④	
		}
		System.out.println("result的值为:"+result);
	}
	/**内部类*/
	private class ReadWriteThread extends Thread{
		//根据构造方法中传入的flag参数,确定线程执行读操作或者写操作
		private boolean flag;
		public ReadWriteThread (boolean flag) {
			this.flag = flag;
		}
		
		@Override
		public void run() {
			if (flag) {
				write();
			}else{
				read();
			}
		}
	}
	/**	启动线程
	 * 	结果为6: ①->②->③->④
	 * 	结果为0:
	 * */
	public static void main(String[] args) {
		SynchronizedDemo demo  = new SynchronizedDemo();
		//启动线程,执行写操作
		demo.new ReadWriteThread(true).start();
		//启动线程,执行写操作
		demo.new ReadWriteThread(false).start();
	}
}

```
## 代码分析
我们在代`write()`方法和`read()`中进行了对共享变量的操作,在多线程执行时,会根据线程抢到的CPU时间片进行操作,也就是说,对于①,②,③,④的操作在不同线程的间的执行顺序是不同的,可能会因为A线程修改过数据后未及时将其写入主内存中,造成B线程操作时数据尚未及时修正,最终出现输出结果的不同
## `synchronized`关键字实现

对于需要修改共享内存方法,我们可以通过`synchronized`关键字修饰,实现对其方法修改变量的一致性维护.

``` java

/**写操作,为共享变量写入最新值*/
	public synchronized void write() {		//synchronized修饰
		ready = true;				//①
		number = 2;					//②
	}
	
	/**读操作,为共享变量读取最新值*/
	public synchronized void read() {
		if (ready) {				//③
			result = number * 3;	//④	
		}
		System.out.println("result的值为:"+result);
	}
	
```

## `volatile` 关键字实现
通过`volatile`修饰共享变量,结合`ReentrantLock()`原子锁,实现对共享方法的调用.

``` java
public class VolatileDemo {

	//声明一个变量
	private volatile int number = 0;
	
	//提供get方法,获取该变量
	public int getNumber() {
		return this.number;
	}
	//原子锁
	private Lock lock = new ReentrantLock();
	
	//对改变变量进行自增操作
	public void inCreate() {		//增加synchronized关键字保证原子性
						//自加分三步:读取->自加->写入,不能保证原子性,在多线程多次写入后,发生抢占数据,值会改变
		//增加锁
		lock.lock();
		try {
			this.number ++;		//执行自增操作
		} finally {
			lock.unlock();		//解锁
		}
	}
	
	
	/**主线程,使用不同的分线程,完成自加操作*/
	public static void main(String[] args) {
		
		final VolatileDemo demo = new VolatileDemo();
		for (int i = 0; i < 500	; i++) {	//执行500次多线程
			//传入匿名内部类,创建实例
			new Thread(new Runnable() {
				@Override
				public void run() {
					demo.inCreate();
				}
			}).start();					//执行线程
		}
		
		/**如果当前仍有子线程在运行,则让主线让出CUP,等待所有子线程运行完后,执行主线程*/
		while(Thread.activeCount() > 1){
			Thread.yield();
		}
		System.out.println("当前变量值为:"+demo.getNumber());
	}
}

```





[1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/armVisible/SynchronizedDemo.java
[2]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/armVisible/VolatileDemo.java

