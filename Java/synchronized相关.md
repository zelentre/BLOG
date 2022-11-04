---
title: synchronized的工作原理
date: 2022-10-24 09:51:09
categories: 
 - java
tags: 
 - synchronized
---

## synchronized的工作原理

### 一、简介

> synchronized是一个同步关键字，在某些多线程场景下，如果不进行同步会导致共享数据不安全，而synchronized关键字就可以用于代码同步。

synchronized主要有三种使用形式：

- 修饰普通同步方法
  - 锁的对象是当前实例对象
- 修饰静态同步方法
  - 锁的对象是当前的类的Class字节码对象
- 修饰同步代码块
  - 锁的对象是 synchronized后面括号里配置的对象，可以是某个对象，也可以是某个类的.class对象

### 二、synchronized的特性

#### 原子性

- 原子性指的是在一次或多次操作中，要么所有的操作都执行并且不会受其他因素干扰而中断，要么所有的操作都不执行。

#### 可见性

- 可见性是指一个线程对共享变量进行修改，另一个线程可以立即读取得到修改后的最新值。

- synchronized可见性是通过内存屏障实现的，按照可见性划分，内存屏障分为：

  - Load屏障：执行refresh，从其他处理器的高速缓冲、主内存，加载数据到自己的高速缓存，保证数据是最新的
  - Store屏障：执行flush操作，自己处理器更新的变量的值，刷新到高速缓存、主内存去

  > 获取锁时，会清空当前线程工作内存中共享变量的副本值，重新从主内存中获取变量的最新的值；
  >
  > 释放锁时，会将工作内存的值重新刷新回主内存

  ```java
  int a = 0;
  synchronize (this){   //monitorenter
      // Load内存屏障
      int b = a;  // 读，通过load内存屏障，强制执行refresh，保证读到最新的
      a = 10; // 写，释放锁时会通过Store，强制flush到高速缓存或主内存
  }    //monitorexit
  //Store内存屏障
  ```

#### 有序性

- 有序性是指程序中代码的执行顺序，Java在编译时和运行时会对代码进行优化，会导致程序最终的执行顺序不一定就是我们编写代码时的顺序。例如，instance = new Singleton()实例化对象的语句分为三步：

  1. 分配对象的内存空间；
  2. 初始化对象；
  3. 设置实例对象指向刚分配的内存地址；

  > 上述第二步操作需要依赖第一步，但是第三步操作不需要依赖第二步，所以执行顺序可能为：1->2->3、1->3->2，当执行顺序为1->3->2时，可能实例对象还没正确初始化，我们直接拿到使用的时候可能会报错。

- synchronized的有序性是依靠内存屏障实现的。按照有序性，内存屏障可分为：

  - Acquire屏障：load屏障之后，加Acquire屏障。它会禁止同步代码块内的读操作，和外面的读写操作发生指令重排；
  - Release屏障：禁止写操作，和外面的读写操作发生指令重排；

- 在 monitorenter 指令和 Load 屏障之后，会加一个 Acquire屏障，这个屏障的作用是禁止同步代码块里面的读操作和外面的读写操作之间发生指令重排，在 monitorexit 指令前加一个Release屏障，也是禁止同步代码块里面的写操作和外面的读写操作之间发生重排序。如下：

  ```java
  int a = 0;
  synchronize (this){  //monitorenter
      // Load内存屏障
      // Acquire屏障，禁止代码块内部的读，和外面的读写发生指令重排
      int b = a;
      a = 10;    //注意：内部还是会发生指令重排
      // Release屏障，禁止写，和外面的读写发生指令重排
  } //monitorexit
  //Store内存屏障
  ```

#### 可重入特性

- 可重入指的就是一个线程可以多次执行synchronized，重复获取同一把锁。举个例子：

  ```java
  public class RenentrantDemo {
      // 锁对象
      private static Object obj = new Object();
  
      public static void main(String[] args) {
          // 自定义Runnable对象
          Runnable runnable = () -> {
              //  使用嵌套的同步代码块
              synchronized (obj) {
                  System.out.println(Thread.currentThread().getName() + "第一次获取锁资源...");
                  synchronized (obj) {
                      System.out.println(Thread.currentThread().getName() + "第二次获取锁资源...");
                      synchronized (obj) {
                          System.out.println(Thread.currentThread().getName() + "第三次获取锁资源...");
                      }
                  }
              }
          };
  
          new Thread(runnable, "t1").start();
      }
  }
  ```

  运行结果：

  ```java
  t1第一次获取锁资源...
  t1第二次获取锁资源...
  t1第三次获取锁资源...
  ```

### 三、synchonized的使用及通过反汇编分析其原理

#### 1. 修饰代码块

```java
public class SynchronizedDemo{
    // 锁对象
    private static Object obj = new Object();
    public static void main(String[] args){
        synchronized(obj){
            System.out.println("execute main()...");
        }
    }
}
```

使用`java -p -v .\SynchronizedDemo.class`命令对字节码进行反汇编，查看字节码指令：

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20221104213047.png)

##### monitorenter指令

> 官网对monitorenter指令的介绍，就是说每一个对象都会和一个监视器对象monitor关联，监视器被占用时会被锁住，其他线程无法来获取该monitor。 当JVM执行某个线程的某个方法内部的monitorenter时，它会尝试去获取当前对象对应的monitor的所有权。大体过程如下：

- 若monior的进入数为0，线程可以进入monitor，并将monitor的进入数置为1，当前线程成为monitor的owner**（拥有这把锁的线程）**；
- 若线程已拥有monitor的所有权，允许它重入monitor，则进入monitor的进入数加1**（记录线程拥有锁的次数）**；
- 若其他线程已经占有monitor的所有权，那么当前尝试获取monitor的所有权的线程会被阻塞，直到monitor的进入数变为0，才能重新尝试获取monitor的所有权；

##### monitorexit指令

> 官网对monitorexit指令的介绍，就是说能执行monitorexit指令的线程一定是拥有当前对象的monitor的所有权的线程；执行monitorexit时会将monitor的进入数减1，当monitor的进入数减为0时，当前线程退出。

**为什么字节码中存在两个monitorexit指令？**

**其实第二个monitorexit指令，是在程序发生异常时候用到的，也就说明了synchronized在发生异常时，会自动释放锁。**

ObjectMonitor对象监视器结构如下：

```java
ObjectMonitor() {
    _header       = NULL;		//锁对象的原始对象头
    _count        = 0;			//抢占当前锁的线程数量
    _waiters      = 0,			//调用wait方法后等待的线程数量
    _recursions   = 0;			//记录锁重入次数
    _object       = NULL;
    _owner        = NULL;		//指向持有ObjectMonitor的线程
    _WaitSet      = NULL;		//处于wait状态的线程队列,等待被唤醒
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ;		//等待锁的线程队列
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
    _previous_owner_tid = 0;
  }
```

#### 2. 修饰普通方法

```java
public class SynchronizedDemo02 {

    public static void main(String[] args) {
    }

    // 修饰普通方法
    public synchronized void add() {
        System.out.println("add...");
    }
}
```

使用`javap -p -v .\SynchronizedDemo02.class`命令对字节码进行反汇编，查看字节码指令：

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20221104231848.png)

如上图，我们可以看到同步方法在反汇编后，不再是通过插入monitorentry和monitorexit指令实现，而是会增加 ACC_SYNCHRONIZED 标识隐式实现的，如果方法表结构（method_info Structure）中的ACC_SYNCHRONIZED标志被设置，那么线程在执行方法前会先去获取对象的monitor对象，如果获取成功则执行方法代码，执行完毕后释放monitor对象，如果monitor对象已经被其它线程获取，那么当前线程被阻塞。

#### 3. 修饰静态方法

```java
public class SynchronizedDemo03 {

    public static void main(String[] args) {
        add();
    }

    // 修饰静态方法
    public synchronized static void add() {
        System.out.println("add...");
    }
}
```

使用`javap -p -v .\SynchronizedDemo03.class`命令对字节码进行反汇编，查看字节码指令：

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20221104233859.png)

### 四、synchronized锁对象存在哪里？

> 之前对对象的内存布局的介绍中，我们知道一个对象，包括对象头、实例数据、对齐填充。而对象头又包括mark  word标记字、类型指针、数组长度(只有数组对象才有)。在mark word标记字中，有一块区域主要存放关于锁的信息。

**存在锁对象的对象头的MarkWord标记字中。如下图：**

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20221104234441.png)

### 五、synchronized与lock的区别？

| synchronized           | lock                                |
| ---------------------- | ----------------------------------- |
| 关键字                 | 接口                                |
| 自动释放锁             | 必须手动调用unlock()方法释放锁      |
| 不能知道线程是否拿到锁 | 可以知道线程是否拿到锁              |
| 能锁住方法和代码块     | 只能锁住代码块                      |
| 读、写操作都堵塞       | 可以使用读锁，提高多线程读效率      |
| 非公平锁               | 通过构造方法可指定是公平锁/非公平锁 |

### 六、总结

1. synchronized修饰代码块的时候，通过在生成的字节码指令中插入monitorenter和monitorexit指令来完成对对象监视器锁的获取和释放；
2. synchronized修饰普通方法和静态方法的时候，通过在字节码中的方法头信息中添ACC_SYNCHRONIZED标识，线程在执行方法前会先去获取对象的monitor对象，如果获取成功则执行方法代码，执行完毕后释放monitor对象；
3. synchronized修饰代码块，锁的对象就是代码块中的对象；修饰普通方法的时候，锁的对象就是当前对象this；修饰静态方法的时候，锁的对象就是当前类的Class字节码对象（类对象）；
4. 使用synchronized修饰实例对象时，如果一个线程正在访问实例对象的一个synchronized方法时，其它线程不仅不能访问该synchronized方法，该对象的其它synchronized方法也不能访问，因为一个对象只有一个监视器锁对象，但是其它线程可以访问该对象的非synchronized方法。
5. 线程A访问实例对象的非static synchronized方法时，线程B也可以同时访问实例对象的static synchronized方法，因为前者获取的是实例对象的监视器锁，而后者获取的是类对象的监视器锁，两者不存在互斥关系。

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20221104235337.png)
