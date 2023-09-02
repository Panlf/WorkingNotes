# Java的知识点

## 线程池
### 线程池创建
Java本身给我们提供了四种线程池`newCachedThreadPool`、`newFixedThreadPool`、`newScheduledThreadPool`和`newSingleThreadExecutor`。

#### 线程池构造参数
在理解Java本身提供的线程池的之前，先了解线程池创建参数。我们可以通过ThreadPoolExecutor来自定义构建一个线程池。
```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

参数说明
- corePoolSize 线程池核心线程数量，核心线程不会被回收，即使没有任务执行，也会保持空闲状态。
- maximumPoolSize 线程池允许创建的最大的线程数，如果队列满了，并且已经创建的线程数小于最大线程数，则线程数会再创建新的线程执行任务。
- keepAliveTime 线程活动时间，超过corePoolSize之后的临时线程的存活时间。
- unit 线程活动时间的单位。
- workQueue 用于保存等待执行的任务的阻塞队列。可以选择ArrayBlockQueue、LinkedBlockingQueue、SynchronousQueue、SynchronousQueue等。
- threadFactory 用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置自定义名称。
- handler 线程池的拒绝策略。系统中的策略有AbortPolicy直接抛出异常、CallerRunsPolicy当前调用者所在线程来执行任务、DiscardOldestPolicy丢弃队列中最老的任务来执行当前任务、DiscardPolicy直接丢弃不处理，当然我们也可以自定义拒绝策略，只要实现RejectedExecutionHandler即可。

#### newCachedThreadPool
```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
可缓存的线程池，从源码中可知其核心线程数量为0，线程最大值可达到Integer的最大值，且空闲线程等待新任务的最大时间为60s，60s后将会销毁。

#### newFixedThreadPool

```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

```
定长线程池，线程池的最大线程数等于核心线程数，并且线程池的线程不会因为闲置超时被销毁。

#### newSingleThreadExecutor
```
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```
单线程池。


#### newScheduledThreadPool
```
public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
```
定时线程池。

### 任务提交
我们可以使用两个方法向线程池提交任务，分别为`execute()`和`submit()`方法。

#### execute()
```
void execute(Runnable command);
```

用于提交不需要返回值的任务。

#### submit()
```
public Future<?> submit(Runnable task) {
            return e.submit(task);
}
public <T> Future<T> submit(Callable<T> task) {
    return e.submit(task);
}
public <T> Future<T> submit(Runnable task, T result) {
    return e.submit(task, result);
}
```
用于提交需要返回值的任务。

### 关闭线程池
可以通过调用线程池的shutdown()或者shutdownNow()方法来关闭线程池。原理就是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程。

```
public void shutdown() { e.shutdown(); }
public List<Runnable> shutdownNow() { return e.shutdownNow(); }
```
更多源码信息可以在`ThreadPoolExecutor`类中搜寻到。

## BigDecimal
### float和double的精度

在使用BigDecimal先了解float和double的精度问题，float占32位(bit),double占64位，即float占4个字节，double占8个字节。

float和double的范围是由指数的位数来决定的。float的存储时候1个bit是符号位，8个bit是指数位，剩下的23个bit是有效数字位，double存储的时候1个bit是符号位，11个bit是位指数位，剩下的52个bit是尾数位。精度由尾数决定，float的精度为7~8位有效数字，double的精度为16~17位。

我们在使用float或者double会出现精度问题
```
float a = 0.05f;
float b = 0.01f;
System.out.println(a + b);
```
这个代码出现的结果不是0.06，而是0.060000002，同样double也会出现这样的问题
```
double c = 0.05d;
double d = 0.01d;
System.out.println(c + d);  
```
打印结果是0.060000000000000005。

基于上述会出现的精度丢失的问题，我们有必要使用BigDecimal进行精确的计算。

### 构造函数
```
BigDecimal(int)
创建一个具有参数所指定整数值的对象

BigDecimal(double)
创建一个具有参数所指定双精度值的对象

BigDecimal(long)
创建一个具有参数所指定长整数值的对象

BigDecimal(String)
创建一个具有参数所指定以字符串表示的数值的对象
```

虽然构造函数支持float、double，但是我们也千万要注意使用string，不然同样会出现精度问题
```
float a = 0.05f;
float b = 0.01f;
BigDecimal m = new BigDecimal(a);
BigDecimal n = new BigDecimal(b);
System.out.println(m.add(n));
```
结果是0.060000000000000005，因为你以为传入a的值是0.05，但是BigDecimal构造的时候不是0.005，而是其更精确的值。这样就还是会出现精度问题，所以要用string方式构造。
```
BigDecimal s = new BigDecimal(String.valueOf(a));
BigDecimal t = new BigDecimal(String.valueOf(b));
System.out.println(s.add(t));
```
结果就是0.06

### 常用方法
```
add(BigDecimal)
BigDecimal对象中的值相加，返回BigDecimal对象

subtract(BigDecimal)
BigDecimal对象中的值相减，返回BigDecimal对象

multiply(BigDecimal)
BigDecimal对象中的值相乘，返回BigDecimal对象

divide(BigDecimal)
BigDecimal对象中的值相除，返回BigDecimal对象

toString()
将BigDecimal对象中的值转换成字符串

doubleValue()
将BigDecimal对象中的值转换成双精度数

floatValue()
将BigDecimal对象中的值转换成单精度数

longValue()
将BigDecimal对象中的值转换成长整数

intValue()
将BigDecimal对象中的值转换成整数
```

## ThreadLocal
### ThreadLocal的概念
ThreadLocal是Thread的局部变量，同一个ThreadLocal所包含的对象，在不同的Thread中有不同的副本。

> ThreadLocal提供了线程内存储变量的能力，这些变量不同之处在于每一个线程读取的变量是对应的互相独立的。通过get和set方法就可以得到当前线程对应的值。

原理是每个ThreadLocal内部都有一个ThreadLocalMap,他保存的key是ThreadLocal的实例，他的值是当前线程的局部变量的副本的值。

### ThreadLocal的方法
- get():返回此线程局部变量当前副本中的值
- set(T value):将线程局部变量当前副本中的值设置为指定值
- initialValue():返回此线程局部变量当前副本中的初始值
- remove():移除此线程局部变量当前副本中的值

### ThreadLocal的问题
ThreadLocal操作不当会引起内存泄露问题，最主要的原因在于它的内部类ThreadLocalMap中的Entry的设计。Entry继承了WeakReference<ThreadLocal<?>>，即Entry的key是弱引用，弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。虽然key会在垃圾回收的时候被回收掉，而key对应的value是强引用，不会被清理， 这样会导致一种现象：key为null，value有值。key为空的话value是无效数据，久而久之，value累加就会导致内存泄漏。

Entry的设计
```
static class ThreadLocalMap {
   static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
...
}
```

ThreadLocalMap实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录。如果说会出现内存泄漏，那只有在出现了 key 为 null 的记录后，没有手动调用 remove() 方法，并且之后也不再调用 get()、set()、remove() 方法的情况下，才会出现内存泄露问题
	

### ThreadLocal的使用场景
ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用，也即变量在线程间隔离而在方法或类间共享的场景。

