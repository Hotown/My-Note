# Java并发编程 —— 浅谈Runnable和Callable

---

Runnable接口：

```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

Callable接口：

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

---

### 异同

#### 相同点

1. 都是接口（=-=|||）
2. 都需要thread.start()来启动线程

#### 不同点

1. 从接口源码注释可以看出，Runnable规定的方法是run()，Callable规定的方法是call()
2. Callable执行后可以返回值，Runnable无返回值
3. call()方法可以throws Exception，run()不能

---

Callable接口的实现类可以通过一个FutureTask.get()来获取call()中的返回值，此方法会阻塞线程知道获得“将来”的结果，不调用此方法，主线程不会阻塞。

---

**最后附上两段抄来的demo**


Callable工作demo：

```java
package com.callable.runnable;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * Created on 2016/5/18.
 */
public class CallableImpl implements Callable<String> {

    public CallableImpl(String acceptStr) {
        this.acceptStr = acceptStr;
    }

    private String acceptStr;

    @Override
    public String call() throws Exception {
        // 任务阻塞 1 秒
        Thread.sleep(1000);
        return this.acceptStr + " append some chars and return it!";
    }


    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Callable<String> callable = new CallableImpl("my callable test!");
        FutureTask<String> task = new FutureTask<>(callable);
        long beginTime = System.currentTimeMillis();
        // 创建线程
        new Thread(task).start();
        // 调用get()阻塞主线程，反之，线程不会阻塞
        String result = task.get();
        long endTime = System.currentTimeMillis();
        System.out.println("hello : " + result);
        System.out.println("cast : " + (endTime - beginTime) / 1000 + " second!");
    }
}
```

Runnable工作demo

```java
package com.callable.runnable;

/**
 * Created on 2016/5/18.
 */
public class RunnableImpl implements Runnable {

    public RunnableImpl(String acceptStr) {
        this.acceptStr = acceptStr;
    }

    private String acceptStr;

    @Override
    public void run() {
        try {
            // 线程阻塞 1 秒，此时有异常产生，只能在方法内部消化，无法上抛
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 最终处理结果无法返回
        System.out.println("hello : " + this.acceptStr);
    }


    public static void main(String[] args) {
        Runnable runnable = new RunnableImpl("my runable test!");
        long beginTime = System.currentTimeMillis();
        new Thread(runnable).start();
        long endTime = System.currentTimeMillis();
        System.out.println("cast : " + (endTime - beginTime) / 1000 + " second!");
    }
}

```



