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
8. largestPoolSize: 是一个动态变量,是记录Poll曾经达到的最高值,也就是 largestPoolSize<= maximumPoolSize.

##### Worker

Worker继承了AQS抽象类并且实现了Runnable接口，是ThreadPoolExecutor的核心内部类。是真正执行线程的。
##### BlockingQueue
BlockingQueue是双缓冲队列。BlockingQueue内部使用两条队列，允许两个线程同时向队列一个存储，一个取出操作。在保证并发安全的同时，提高了队列的存取效率。
- ArrayBlockingQueue（int i）
- LinkedBlockingQueue（）
- PriorityBlockingQueue（）
- SynchronizedQueue（）

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
### Executors
executors提供几种常用的线程池，方便使用，但最好不要使用，因为很有可能提供的几种并不是你使用场景里面较优的方案
#### newSingleThreadExecutor
创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。
#### newFixedThreadPool
创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。
#### newCachedThreadPool
创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，
那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。
#### newScheduledThreadPool
创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。
