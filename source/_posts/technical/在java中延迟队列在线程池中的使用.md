---
cover:
title: java中延迟队列在线程池中的使用
date: 2025-11-28
categories:
  - 技术
  - 后端
tags:
  - 笔记
---

# java中延迟队列在线程池中的使用

## 延迟队列——DelayQueue

是一种特殊的队列，它允许我们在指定延迟时间之后弹出元素，以便我们执行任务。

学习DelayQueue的时候，需要注意需要先去了解TimeUnit类，这个是时间单位类，可以基于时间单位对时间戳操作。

DelayQueue中的队列必须要实现Delayed接口

```java
package testLock;

import java.util.Date;
import java.util.concurrent.Delayed;
import java.util.concurrent.TimeUnit;

/**
 * @BelongsProject: LockDemo
 * @BelongsPackage: testLock
 * @Author: hanhaitao
 * @CreateTime: 2025-11-27  11:51
 * @Description: TODO
 * @Version: 1.0
 */
public class DelayTask implements Delayed, Runnable {

    private final String taskName;
    //执行的时间
    private final long executeTime; // 执行时间戳

    public DelayTask(String taskName, long delay, TimeUnit unit) {
        this.taskName = taskName;
        this.executeTime = System.currentTimeMillis() + unit.toMillis(delay);
    }
    @Override
    public long getDelay(TimeUnit unit) {
       	//这的代码的意思是，执行的时间还剩多久，当剩余时间小于零时，弹出队列。
        return unit.convert(executeTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }

    @Override
    public int compareTo(Delayed other) {
        //给两个元素在队列里面排序使用
        return Long.compare(this.executeTime, ((DelayTask) other).executeTime);
    }



    @Override
    public void run() {
        System.out.println("执行延迟任务: " + taskName + ", 时间: " + new Date());
        try {
            Thread.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

## 在线程池中使用

```java
package testLock;

import java.time.LocalDateTime;
import java.util.Date;
import java.util.concurrent.*;

/**
 * @BelongsProject: LockDemo
 * @BelongsPackage: testLock
 * @Author: hanhaitao
 * @CreateTime: 2025-11-27  11:46
 * @Description:
 * @Version: 1.0
 */
public class DelayQueuePoolDemo {

    public ExecutorService  executor;

    public DelayQueuePoolDemo(){
        ArrayBlockingQueue<DelayTask> delayTasks = new ArrayBlockingQueue<>(5);
        //延迟队列是不阻塞队列，不可以作为线程的参数。
        executor = new ThreadPoolExecutor(1,2,20, TimeUnit.SECONDS,new ArrayBlockingQueue<>(5));
//        executor = Executors.newFixedThreadPool(1);
    }

    public static void main(String[] args) {
        DelayQueuePoolDemo delayQueuePoolDemo = new DelayQueuePoolDemo();
        ExecutorService service = delayQueuePoolDemo.executor;
        DelayQueue<DelayTask> delayQueue = new DelayQueue<>();
        delayQueue.add(new DelayTask("task1", 1000, TimeUnit.MILLISECONDS));
        delayQueue.add(new DelayTask("task2", 2000, TimeUnit.MILLISECONDS));
        delayQueue.add(new DelayTask("task3", 500, TimeUnit.MILLISECONDS));
        delayQueue.add(new DelayTask("task4", 1500, TimeUnit.MILLISECONDS));
        long start = System.currentTimeMillis();
        for(int i = 0; i < 4; i++){
            try {
                //会等待队列弹出元素
                DelayTask take = delayQueue.take();
                service.submit(take);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        boolean shutdown = service.isShutdown();
        System.out.println("shutdown = " + shutdown);
        service.shutdown();
        long end = System.currentTimeMillis();
        System.out.println(end-start);
    }
}
```

按照延迟时间依次执行任务

执行结果：

```java
执行延迟任务: task3, 时间: Thu Nov 27 23:44:52 CST 2025
执行延迟任务: task1, 时间: Thu Nov 27 23:44:52 CST 2025
执行延迟任务: task4, 时间: Thu Nov 27 23:44:53 CST 2025
执行延迟任务: task2, 时间: Thu Nov 27 23:44:53 CST 2025
shutdown = false
2004
```
