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

