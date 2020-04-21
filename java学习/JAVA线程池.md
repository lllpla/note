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

