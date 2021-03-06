# Linux内核学习（三）—— 内核同步介绍

## 临界区和竞争条件

**临界区：**访问和操作共享数据的代码段。

**竞争条件：**两个执行线程处于同一临界区运行的情况。

多个执行线程并发访问同一资源往往存在安全隐患，为了避免在临界区中并发访问，我们必须保证这样的代码原子地执行。

避免并发和防止竞争条件称为同步（synchronization）。

## 加锁

为一个临界区增加保护。当一个执行线程进入空的临界区时，它会获得该临界区的锁，然后正常执行。此时，如果有另一个执行线程想要进入该临界区，那么它就会强制等待到之前的线程执行完毕，直到上一个执行线程释放锁，该执行进程才能进去临界区执行。

### 造成并发执行的原因

+ 中断 —— 中断几乎可以在任何时刻异步发生，也可以随时打断当前正在执行的代码。
+ 软中断和tasklet
+ 内核抢占 —— 内核的抢占性导致，内核中的任务可能会被另一任务抢占。
+ 睡眠及与用户空间的同步 —— 在内核执行的进程可能会睡眠，这就会唤醒调度程序，从而调度一个新的用户进程执行。
+ 对称多处理 —— 两个或多个处理器可以同时执行代码。

## 死锁

### 死锁的产生

有一个或多个执行线程和一个或多个资源，每个线程都在等在其中的一个资源，但是所有的资源都已经被占用。所有的线程都相互等待，导致它们永远不会释放锁。

### 避免死锁的规则

+ 按顺序加锁。使用嵌套锁时，必须保证以相同的顺序获取锁。
+ 防止发生饥饿。
+ 不要重复请求同一个锁。
+ 设计应力求简单。

## 争用和扩展性

**锁的争用**（lock contention），简称**争用**，是指锁正在被占用时，有其他的线程试图获得锁。

一个锁处于**高度争用状态**是指，有多个其他线程正在等待获得锁。

由于锁的作用是使程序以串行的方式对资源进行访问，过度使用锁无疑会降低系统的性能。被高度争用的锁会成为系统的瓶颈，严重降低系统性能。

**扩展性**（scalability）是对系统可扩展程度的一个度量。


