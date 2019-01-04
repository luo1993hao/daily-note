### ThreadPoolExecutor

继承 `AbstractExecutorService`, 是 `ExecutorService` 的抽象实现。

#### ThreadPoolExecutor 结构

ThreadPoolExecutor 中包含一个继承 AQS 的封装的 Runnable、4种拒绝策略以及线程管理的具体实现。

- 核心属性

1. ctl： 线程池的控制状态（用来表示线程池的运行状态，整型的高3位）和运行的worker数量（低29位）
2. corePoolSize： 线程池核心大小，正在运行的线程数小于等于coreSize
3. maximumPoolSize：最大大小
4. workQueue：阻塞队列
5. keepAliveTime：排队等待时间
6. threadFactory：用来对线程池中的线程命名
7. handler：拒绝策略

##### Worker

Worker继承了AQS抽象类并且实现了Runnable接口，是ThreadPoolExecutor的核心内部类。是真正执行线程的。

##### 拒绝策略

根据传入的拒绝策略，处理排队失败的线程。

1. CallerRunsPolicy: 根据传入的线程处理。
2. AbortPolicy： 抛出异常
3. DiscardPolicy： 丢弃
4. DiscardOldestPolicy： 丢弃最老的并重试

##### 构造方法

```$xslt
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

##### 管理方法

- execute

1. 如果运行的线程小于corePoolSize,则尝试使用用户定义的Runnalbe对象创建一个新的线程调用addWorker函数会原子性的检查runState和workCount，通过返回false来防止在不应该添加线程时添加了线程。
2. 如果一个任务能够成功入队列，在添加一个线城时仍需要进行双重检查（因为在前一次检查后该线程死亡了），或者当进入到此方法时，线程池已经shutdown了，所以需要再次检查状态，若有必要，当停止时还需要回滚入队列操作，或者当线程池没有线程时需要创建一个新线程
3. 如果无法入队列，那么需要增加一个新线程，如果此操作失败，那么就意味着线程池已经shutdown或者已经饱和了，所以拒绝任务

- addWorker

1. 原子性的增加workerCount。
2. 将用户给定的任务封装成为一个worker，并将此worker添加进workers集合中。
3. 启动worker对应的线程，并启动该线程，运行worker的run方法。
4. 回滚worker的创建动作，即将worker从workers集合中删除，并原子性的减少workerCount。

- runWorker

此函数中会实际执行给定任务（即调用用户重写的run方法），并且当给定任务完成后，会继续从阻塞队列中取任务，直到阻塞队列为空（即任务全部完成）。在执行给定任务时，会调用钩子函数，利用钩子函数可以完成用户自定义的一些逻辑。在runWorker中会调用到getTask函数和processWorkerExit钩子函数。

- submit

直接调用 execute。

- shutdown

关闭线程池，等待执行。

- shutdownNow

直接关闭线程池，返回未执行的线程。

shutdown 和 shutdownNow 方法中执行流程的不同有：
1. shutdown 中改变状态为 `SHUTDOWN`， shutdownNow 中改变状态为 `STOP`
2. shutdownNow 中会取出阻塞队列中的Worker返回。
