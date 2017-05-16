---
title: Java 8 多线程基础
tags: [Java]
categories: [Java]
photo:
  - /images/java8.jpg
date: 2017-04-24 21:04:32
---
Java Concurrency API 从Java 5开始引入，到现在做了很多改进。

## Part 1: Threads and Executors

现在操作系统通过进程和线程实现并发。进程拥有独立的资源，一个进程中可有创建多个线程，折线线程共享进程的资源。

### **Thread**
```java
Runnable task = () -> {
    String threadName = Thread.currentThread().getName();
    System.out.println("Hello " + threadName);
};

task.run();

Thread thread = new Thread(task);
thread.start();

System.out.println("Done!");
```
output
```
Hello main
Hello Thread-0
Done!
```

线程可以sleep
```java
TimeUnit.SECONDS.sleep(1);//暂停1秒
```
<!--more-->
### **Executors**

Java Concurrency API 引入`ExecutorService`作为一个更高级的创建线程的方法。
`Executors可以创建能够运行异步任务的线程池。所有我们不需要手动创建线程了。线程池中的线程可以重复使用。

example
```java
ExecutorService executor = Executors.newSingleThreadExecutor();//只有一个线程的池
executor.submit(() -> {
    String threadName = Thread.currentThread().getName();
    System.out.println("Hello " + threadName);
});

// => Hello pool-1-thread-1
```

线程池并不会自己关闭。`ExecutorService`提供两个方法：
`shutdown()`:等待当前运行的线程结束。
`shutdownNow()`:立即中断所有线程并关闭线程池。

```java
try {
    System.out.println("attempt to shutdown executor");
    executor.shutdown();
    executor.awaitTermination(5, TimeUnit.SECONDS);
}
catch (InterruptedException e) {
    System.err.println("tasks interrupted");
}
finally {
    if (!executor.isTerminated()) {
        System.err.println("cancel non-finished tasks");
    }
    executor.shutdownNow();
    System.out.println("shutdown finished");
}
```

### **Callables and Futures**

`Callable`的功能和`Runnable`一样，只是有返回值。
```java
Callable<Integer> task = () -> {
    try {
        TimeUnit.SECONDS.sleep(1);
        return 123;
    }
    catch (InterruptedException e) {
        throw new IllegalStateException("task interrupted", e);
    }
};
```
`Callable`也可以使用`ExecutorService`提交，但是必须使用`Future`接受返回值。

```java
ExecutorService executor = Executors.newFixedThreadPool(1);
Future<Integer> future = executor.submit(task);

System.out.println("future done? " + future.isDone());

Integer result = future.get();

System.out.println("future done? " + future.isDone());
System.out.print("result: " + result);
```
output:
```
future done? false
future done? true
result: 123
```

调用`futer.get()`方法会阻塞，直到callable结束。
为了防止因为阻塞而失去响应，我们可以使用Timeout。

```java
ExecutorService executor = Executors.newFixedThreadPool(1);

Future<Integer> future = executor.submit(() -> {
    try {
        TimeUnit.SECONDS.sleep(2);//等待2秒
        return 123;
    }
    catch (InterruptedException e) {
        throw new IllegalStateException("task interrupted", e);
    }
});

future.get(1, TimeUnit.SECONDS);//等待1秒
```
超时会抛出异常
+
```java
Exception in thread "main" java.util.concurrent.TimeoutException
    at java.util.concurrent.FutureTask.get(FutureTask.java:205)
```

Executors支持使用`invokeAll()`,支持`批量提交`callable任务。
```java
ExecutorService executor = Executors.newWorkStealingPool();

List<Callable<String>> callables = Arrays.asList(
        () -> "task1",
        () -> "task2",
        () -> "task3");

executor.invokeAll(callables)
    .stream()
    .map(future -> {
        try {
            return future.get();
        }
        catch (Exception e) {
            throw new IllegalStateException(e);
        }
    })
    .forEach(System.out::println);
```

`invokeAny()` 返回`最先`执行Callable的值。
```java
Callable<String> callable(String result, long sleepSeconds) {
    return () -> {
        TimeUnit.SECONDS.sleep(sleepSeconds);
        return result;
    };
}

ExecutorService executor = Executors.newWorkStealingPool();

List<Callable<String>> callables = Arrays.asList(
    callable("task1", 2),
    callable("task2", 1),
    callable("task3", 3));

String result = executor.invokeAny(callables);
System.out.println(result);

// => task2
```
`newWorkStealingPool()`使用了`ForkJoinPools`，默认的并行数=CPU的核数。

### **Scheduled Executors**

ScheduledExecutorService可以定时运行任务。

example
```java
ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

Runnable task = () -> System.out.println("Scheduling: " + System.nanoTime());
//延迟3秒执行
ScheduledFuture<?> future = executor.schedule(task, 3, TimeUnit.SECONDS);

TimeUnit.MILLISECONDS.sleep(1337);

long remainingDelay = future.getDelay(TimeUnit.MILLISECONDS);
System.out.printf("Remaining Delay: %sms", remainingDelay);
```

`scheduleAtFixedRate()` 和 `scheduledWidhtFixedDelay()`
```java
ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

Runnable task = () -> System.out.println("Scheduling: " + System.nanoTime());

int initialDelay = 0;
int period = 1;
executor.scheduleAtFixedRate(task, initialDelay, period, TimeUnit.SECONDS);
```
`scheduleAtFixedRate()`的时间间隔并不会包括任务执行的时间。
`scheduleWithFixedDelay()`的时间间隔是上一次结束时间到下一次开始时间的间隔。适合你无法估计任务运行的时间，但是又希望间相同的时间进行。

---

## Part 2 Synchronization and Locks

以下可能使用到的`stop()`和`sleep()`的方法定义：
```java
public class ConcurrentUtils {

    public static void stop(ExecutorService executor) {
        try {
            executor.shutdown();
            executor.awaitTermination(60, TimeUnit.SECONDS);
        }
        catch (InterruptedException e) {
            System.err.println("termination interrupted");
        }
        finally {
            if (!executor.isTerminated()) {
                System.err.println("killing non-finished tasks");
            }
            executor.shutdownNow();
        }
    }

    public static void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (InterruptedException e) {
            throw new IllegalStateException(e);
        }
    }

}
```

### **Synchronized**

定义给一个`increment()`方法，我们需要多个线程同时访问它。
```java
int count = 0;

void increment() {
    count = count + 1;
}
```
```java
ExecutorService executor = Executors.newFixedThreadPool(2);

IntStream.range(0, 10000)
    .forEach(i -> executor.submit(this::increment));

stop(executor);

System.out.println(count);  // 9965
```
increment()操作需要三步：

1. 读取变量
2. 增加1
3. 写回变量

所以我们需要互斥访问这个方法，使用`synchronized` `同步方法`。

```java
synchronized void incrementSync() {
    count = count + 1;
}
```
或者 `同步块`
```java
void incrementSync() {
    synchronized (this) {
        count = count + 1;
    }
}
```
synchronized内部实现是通过`monitor`，每一个对象都有monitor，多个线程访问同一个对象需要获得monitor。
这也解释了`可重入锁`（同步方法调用另一个同步方法，使用同一个对象锁）的原因，一个线程可以安全的多次获取同一把锁。

### **Locks**

**ReentrantLock** (重入锁)
可重入锁与synchronized一样可以实现资源的互斥访问。

```java
ReentrantLock lock = new ReentrantLock();
int count = 0;

void increment() {
    lock.lock();
    try {
        count++;
    } finally {
        lock.unlock();
    }
}
```

**ReadWriteLock**(读写锁)
多个线程同时读不需要加锁，基于一般读的频率大于写的频率，可以提高性能。
```java
ExecutorService executor = Executors.newFixedThreadPool(2);
Map<String, String> map = new HashMap<>();
ReadWriteLock lock = new ReentrantReadWriteLock();

executor.submit(() -> {
    lock.writeLock().lock();
    try {
        sleep(1);
        map.put("foo", "bar");
    } finally {
        lock.writeLock().unlock();
    }
});
```
```java
Runnable readTask = () -> {
    lock.readLock().lock();
    try {
        System.out.println(map.get("foo"));
        sleep(1);
    } finally {
        lock.readLock().unlock();
    }
};

executor.submit(readTask);
executor.submit(readTask);

stop(executor);
```

执行上面的代码，两个读线程必须等待写线程释放写锁，才能执行。这两个读线程是同时执行的，因为资源没有写锁，读锁就可以安全的获得。

### Semaphores

锁用于互斥访问资源或变量，信号量是一个允许访问的集合。
```java
ExecutorService executor = Executors.newFixedThreadPool(10);

Semaphore semaphore = new Semaphore(5);

Runnable longRunningTask = () -> {
    boolean permit = false;
    try {
        permit = semaphore.tryAcquire(1, TimeUnit.SECONDS);
        if (permit) {
            System.out.println("Semaphore acquired");
            sleep(5);
        } else {
            System.out.println("Could not acquire semaphore");
        }
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    } finally {
        if (permit) {
            semaphore.release();
        }
    }
}

IntStream.range(0, 10)
    .forEach(i -> executor.submit(longRunningTask));

stop(executor);
```
output:
```
Semaphore acquired
Semaphore acquired
Semaphore acquired
Semaphore acquired
Semaphore acquired
Could not acquire semaphore
Could not acquire semaphore
Could not acquire semaphore
Could not acquire semaphore
Could not acquire semaphore
```

## Part 3 Atomic Variables and ConcurrentMap

### **AtomicInteger**

`java.concurrent.atomic`包含了很多有用的原子操作。原子级的操作不需要使用`synchronized`或`Lock`就可以并发访问。
原子级操作内部的实现是`compare-an-swap`(CAS)，现代CPU大多都支持CAS的原子级指令。这些指令比`synchronizing`和`Lock`都要高效。
当需要并发频繁修改一个变量时，推荐使用原子操作。

incrementAndGet
```java
AtomicInteger atomicInt = new AtomicInteger(0);

ExecutorService executor = Executors.newFixedThreadPool(2);

IntStream.range(0, 1000)
    .forEach(i -> executor.submit(atomicInt::incrementAndGet));

stop(executor);

System.out.println(atomicInt.get());    // => 1000
```
updateAndGet
```java
AtomicInteger atomicInt = new AtomicInteger(0);

ExecutorService executor = Executors.newFixedThreadPool(2);

IntStream.range(0, 1000)
    .forEach(i -> {
        Runnable task = () ->
            atomicInt.updateAndGet(n -> n + 2);
        executor.submit(task);
    });

stop(executor);

System.out.println(atomicInt.get());    // => 2000
```
accumulateAndGet
```java
AtomicInteger atomicInt = new AtomicInteger(0);

ExecutorService executor = Executors.newFixedThreadPool(2);

IntStream.range(0, 1000)
    .forEach(i -> {
        Runnable task = () ->
            atomicInt.accumulateAndGet(i, (n, m) -> n + m);
        executor.submit(task);
    });

stop(executor);

System.out.println(atomicInt.get());    // => 499500
```
其他原子类`AtomicBoolean` `AtomicLong` `AtomicReference`

### **ConcurrentMap**

example
```java
ConcurrentMap<String, String> map = new ConcurrentHashMap<>();
map.put("foo", "bar");
map.put("han", "solo");
map.put("r2", "d2");
map.put("c3", "p0");
```
```java
map.forEach((key, value) -> System.out.printf("%s = %s\n", key, value));

String value = map.putIfAbsent("c3", "p1");
System.out.println(value);    // p0

String value = map.getOrDefault("hi", "there");
System.out.println(value);    // there

map.replaceAll((key, value) -> "r2".equals(key) ? "d3" : value);
System.out.println(map.get("r2"));    // d3

map.compute("foo", (key, value) -> value + value);
System.out.println(map.get("foo"));   // barbar

map.merge("foo", "boo", (oldVal, newVal) -> newVal + " was " + oldVal);
System.out.println(map.get("foo"));   // boo was foo
```
`ConcurrentHashMap`使用`ForkJoinPool`实现并发。并发的数量取决于CPU的内核数。
```java
System.out.println(ForkJoinPool.getCommonPoolParallelism());  // 3
```
可以修改JVM参数
```
-Djava.util.concurrent.ForkJoinPool.common.parallelism=5
```

**map.search()**
```java
String result = map.search(1, (key, value) -> {
    System.out.println(Thread.currentThread().getName());
    if ("foo".equals(key)) {
        return value;
    }
    return null;
});
System.out.println("Result: " + result);

// ForkJoinPool.commonPool-worker-2
// main
// ForkJoinPool.commonPool-worker-3
// Result: bar
```

**map.redue()**
```java
String result = map.reduce(1,
    (key, value) -> {
        System.out.println("Transform: " + Thread.currentThread().getName());
        return key + "=" + value;
    },
    (s1, s2) -> {
        System.out.println("Reduce: " + Thread.currentThread().getName());
        return s1 + ", " + s2;
    });

System.out.println("Result: " + result);

// Transform: ForkJoinPool.commonPool-worker-2
// Transform: main
// Transform: ForkJoinPool.commonPool-worker-3
// Reduce: ForkJoinPool.commonPool-worker-3
// Transform: main
// Reduce: main
// Reduce: main
// Result: r2=d2, c3=p0, han=solo, foo=bar
```

英文原文：[Java 8 Concurrency Tutorial](http://winterbe.com/posts/2015/04/07/java8-concurrency-tutorial-thread-executor-examples/)