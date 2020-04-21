# JAVA线程池
## 什么是线程池

线程池维护着多个线程，等待监督管理者分配可并发执行的任务。着避免了在处理短时间任务时创建与销毁线程的代价。

```java
start() //创建一定梳理的线程，进行线程循环
stop() //停止所有线程循环，回收所有资源
addTask() //添加任务
```

Excutors创建线程池便捷方法如下：

```java
Executors.newFixedThreadPool(100); //创建一个固定大小的线程池
Executors.newSingleThreadExecutor(); //创建只有一个线程的线程池
Executors.newCachedThreadPool();//创建一个不限线程数上限的线程池，任何提交的任务都将立即执行
```

对于服务端需要长期运行的程序，创建线程池应该使用`ThreadPoolExecutor`的构造方法

```java
public ThreadPoolExecutor(
    int corePoolSize, //线程池长期维持的线程数
    int maximumPoolSize,//线程数上限
    long keepAliveTime, //空闲线程存活时间
    TimeUnit unit, //时间单位
    BlockingQueue<Runnable> workQueue, //任务队列
    ThreadFactory threadFactory,//新线程的产生方式
    RejectedExecutionHandler handler //拒绝策略
)
```

java线程池有**7大参数、4大特性**：

- 特性一：当池中正在运行的线程数(包括空闲线程)小于corePoolSize时，新建线程执行任务。
- 特性二：当池中正在运行的线程数大于等于corePoolSize时，新插入的任务进入workQueue排队(如果workQueue长度允许)，等待空闲线程执行。
- 特性三：当队列中任务数达到上限，并且池中正在运行的线程数小于maximumPoolSize，新加入任务时，新建线程。
- 特性四，当队列中任务数达到上限，并且池中正在运行的线程数等于maximumPoolSize，对新加入的任务，执行拒绝策略(线程池默认拒绝策略为抛异常)

## 种类

### 1. newCachedThreadPool

核心线程数为0，最大线程数为Integer.MAX_VALUE

```java
public static ExecutorService newCachedThreadPool() {
	return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, 
                                  TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

#### 1.1 作用

创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们，并在需要时使用提供的 ThreadFactory 创建新线程。

#### 1.2 特征

- 线程池中数量没有固定，可达到最大值（Interger. MAX_VALUE）
- 线程池中的线程可进行缓存重复利用和回收（回收默认时间为1分钟）
- 当线程池中，没有可用线程，会重新创建一个线程

#### 1.3 创建方式

`Executors.newCachedThreadPool();`

### 2. newFixedThreadPool

核心线程数与最大线程数均为指定的nThreads，空闲线程的存活时间是0，工作队列是无界队列。

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
	return new ThreadPoolExecutor(nThreads, nThreads,
	                                      0L, TimeUnit.MILLISECONDS,
	                                      new LinkedBlockingQueue<Runnable>());
}
```

#### 2.1 作用

创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。在任意点，在大多数 nThreads 线程会处于处理任务的活动状态。如果在所有线程处于活动状态时提交附加任务，则在有可用线程之前，附加任务将在队列中等待。如果在关闭前的执行期间由于失败而导致任何线程终止，那么一个新线程将代替它执行后续的任务（如果需要）。在某个线程被显式地关闭之前，池中的线程将一直存在。

#### 2.2 特征

（1）线程池中的线程处于一定的量，可以很好的控制线程的并发量

（2）线程可以重复被使用，在显示关闭之前，都将一直存在

（3）超出一定量的线程被提交时候需在队列中等待

#### 2.3 创建方式

```java
（1）Executors.newFixedThreadPool(int nThreads)；//nThreads为线程的数量 
（2）Executors.newFixedThreadPool(int nThreads，ThreadFactory threadFactory)；//nThreads为线程的数量，threadFactory创建线程的工厂方式
```

### 3. newSingleThreadExecutor

核心线程数与最大线程数均为1，工作队列是无界队列

```java
public static ExecutorService newSingleThreadExecutor() {
	return new FinalizableDelegatedExecutorService
	            (new ThreadPoolExecutor(1, 1,
	                                    0L, TimeUnit.MILLISECONDS,
	                                    new LinkedBlockingQueue<Runnable>()));
}
```

#### 3.1 作用

创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。（注意，如果因为在关闭前的执行期间出现失败而终止了此单个线程，那么如果需要，一个新线程将代替它执行后续的任务）。可保证顺序地执行各个任务，并且在任意给定的时间不会有多个线程是活动的。与其他等效的 newFixedThreadPool(1) 不同，可保证无需重新配置此方法所返回的执行程序即可使用其他的线程。

#### 3.2 特征

线程池中最多执行1个线程，之后提交的线程活动将会排在队列中以此执行。

#### 3.3 创建方式

```java
Executors.newSingleThreadExecutor() ； 
Executors.newSingleThreadExecutor(ThreadFactory threadFactory)；// threadFactory创建线程的工厂方式
```

### 4. newScheduledThreadPool

指定核心线程数corePoolSize、最大线程数是Integer.MAX_VALUE

DelayedWorkQueue：任务队列会根据任务延时时间的优先级进行执行

```java
public class ScheduledThreadPoolExecutor
        extends ThreadPoolExecutor
        implements ScheduledExecutorService {
	....................
	/**
     * Creates a new {@code ScheduledThreadPoolExecutor} with the
     * given core pool size.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @throws IllegalArgumentException if {@code corePoolSize < 0}
     */
	public ScheduledThreadPoolExecutor(int corePoolSize) {
		super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
		              new DelayedWorkQueue());
	}
```



#### 4.1 作用

创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。

#### 4.2特征

（1）线程池中具有指定数量的线程，即便是空线程也将保留

（2）可定时或者延迟执行线程活动

#### 4.3创建方式

```java
Executors.newScheduledThreadPool(int corePoolSize);// corePoolSize线程的个数 
newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory);// corePoolSize线程的
```

### 5. newSingleThreadScheduledExecutor

#### 5.1 作用

创建一个单线程执行程序，它可安排在给定延迟后运行命令或者定期地执行。

#### 5.2 特征

- 线程池中最多执行1个线程，之后提交的线程活动将会排在队列中以此执行

- 可定时或者延迟执行线程活动

#### 5.3 创建方式

```java
Executors.newSingleThreadScheduledExecutor();
Executors.newSingleThreadScheduledExecutor(ThreadFactory threadFactory);//threadFactory创建线程
```

### 工作队列

### SynchronousQueue：直接提交

是工作队列的默认选项，将任务直接提交给线程而不保持。

如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。

**优缺点**

优点：可以避免在处理可能具有内部依赖性的请求集时出现锁。

直接提交通常要求无界maximumPoolSizes 以避免拒绝新提交的任务。

当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

### ArrayBlockingQueue：无界队列

在所有核心线程都忙时，新任务在队列中等待。

因此，仅创建corePoolSize线程即可。（maximumPoolSize的值没有任何作用。）

当每个任务完全独立于其他任务时，因此任务不会影响彼此的执行。

**优缺点**

优点：例如，在网页服务器中，尽管这种排队方式对于消除短暂的突发请求很有用。

缺点：当命令请求到达速度比其处理速度更快时，工作队列无限制增长。

### LinkedBlockingQueue：有界队列

当与有限的maximumPoolSizes一起使用时，有界队列有助于防止资源耗尽，但是调整和控制起来会更加困难。

队列大小和最大池大小可以相互权衡

使用大队列和小池可以最大程度地减少CPU使用率，操作系统资源和上下文切换开销，但可能导致人为地降低吞吐量。

如果任务频繁阻塞（例如如果它们是受I/O约束的），则系统可能可以为非预定的更多线程安排时间。

使用小队列通常需要更大的池大小，这会使CPU繁忙，但可能会遇到不可接受的调度开销，这也会降低吞吐量。