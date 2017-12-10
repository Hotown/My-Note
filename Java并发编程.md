# Java并发编程

## 线程的生命周期

> 线程的生命周期一共有5种状态：new、runnable、running、blocked、dead.

### 新建(new Thread)

实例化一个Thread类的对象，此线程即进入new状态，分配有自己的内存空间，但该线程并没有运行，此时线程not alive。

### 就绪(runnable)

线程已经被启动，在等待被分配给CPU时间片。也就通过调用线程实例的start()方法来启动线程，进入runnbale。runnable状态的线程已经具备了运行条件，但还没有被分到CPU，不一定会被立即执行，线程处于就绪队列中。此时线程alive。

### 运行(running)

线程获得CPU资源正在执行任务(run()方法)，除非此线程自动放弃CPU资源或者有优先级更高的线程进入，线程将一直运行到结束。线程alive。

### 堵塞(blocked)

由于某种原因导致正在运行的线程让出CPU并暂停自己的执行，即进入阻塞状态。

+ 正在睡眠：用sleep(long t)方法。一个睡眠的进程在指定的时间过去可进入就绪状态。
+ 正在等待：调用wait()。可用notify()回到就绪状态。
+ 被另一个线程所阻塞：调用suspend()方法。可调用resume()回复。

此时线程alive。

### 死亡(dead)

线程执行完毕或被其他线程杀死。此时线程不可能再进入就绪状态等待执行。

+ 自然终止：正常运行run()方法后终止。
+ 异常终止：调用stop()方法让一个线程终止运行

处于Dead状态的线程not alive。

## 守护线程

运行在后台的线程。如Java的垃圾回收，数据库连接池等都是守护线程。

守护线程与普通线程在写法上没有太大区别，调用线程对象的方法setDaemon(true)就可以将其设置为守护线程。

`public final void setDaemon(boolean on) //该方法碧玺在启动线程前调用`

当正在运行的线程都是守护线程时，Java虚拟机退出。即，Java虚拟的运行只关系非守护线程，当非守护线程都运行完毕时，无论守护线程是否退出，Java虚拟机都会停止。

## 当前线程副本：ThreadLocal

ThreadLocal为每个使用该变量的线程提供独立的变量副本，每一个线程都可以独立地改变自己的副本，而不会影响其他线程所对应的副本。

### 实现原理

ThreadLocal的set()方法源码：

```java
    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

分析它的实现：首先通过getMap(Thread t)方法获取一个和当前线程相关的ThreadLocalMap，然后将变量的值设置到这个ThreadLocalMap对象中，若`map == null`，则新建一个ThreadLocalMap然后再赋值。

### 线程隔离的秘密

ThreadLocalMap是ThreadLocal类的一个静态内部类，实现了键值对的设置和获取。每个线程中都有一个独立的ThreadLocalMap副本，它所存储的值，只能被当前线程读取和修改。
ThreadLocal类通过操作每一个线程特有的ThreadLocalMap副本，从而实现了变量访问在不同线程中的隔离。
ThreadLocalMap存储的键值对中的键是this对象指向的ThreadLocal对象。

## Thread安全

### 隐式锁 synchronized线程同步

### Lock和ReentrantLock




