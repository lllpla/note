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
    int corePoolSize, //线程池长期wei'z
)
```



