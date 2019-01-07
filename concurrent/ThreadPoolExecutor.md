
### ThreadPoolExecutor

继承 `AbstractExecutorService`, 是 `ExecutorService` 的抽象实现。
#### 整体流程

![](https://i.loli.net/2019/01/07/5c32ab57499a7.png)

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
- ArrayBlockingQueue（）
 - 是一个用数组实现的有界阻塞队列，此队列按照先进先出（FIFO）的原则对元素进行排序。支持公平锁和非公平锁
- LinkedBlockingQueue（）
 - 一个由链表结构组成的有界队列，此队列的长度为Integer.MAX_VALUE。此队列按照先进先出的顺序进行排序
- PriorityBlockingQueue（）
  - 一个支持线程优先级排序的无界队列，默认自然序进行排序，也可以自定义实现compareTo()方法来指定元素排序规则，不能保证同优先级元素的顺序。
- SynchronousQueue（）
  - 无界，无缓冲的等待队列。其中每个插入操作必须等待另一个线程的对应移除操作 ，反之亦然。同步队列没有任何内部容量，甚至连一个队列的容量都没有
  ，在线程池里的一个典型应用是Executors.newCachedThreadPool()就使用了SynchronousQueue
- 排队有三种通用策略：

   - 直接提交。工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

   - 无界队列。使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

   - 有界队列。当使用有限的 maximumPoolSizes时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。  
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

##### 核心方法（结合源码）

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
#### 线程池监控
- taskCount：线程池需要执行的任务数量。
- completedTaskCount：线程池在运行过程中已完成的任务数量，小于或等于taskCount。
- largestPoolSize：线程池里曾经创建过的最大线程数量。通过这个数据可以知道线程池是否曾经满过。如该数值等于线程池的最大大小，则表示线程池曾经满过。
- getPoolSize：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销毁，所以这个大小只增不减。
- getActiveCount：获取活动的线程数。

通过扩展线程池进行监控。可以通过继承线程池来自定义线程池，重写线程池的beforeExecute、afterExecute和terminated方法，也可以在任务执行前、执行后和线程池关闭前执行一些代码来进行监控。例如，监控任务的平均执行时间、最大执行时间和最小执行时间等
####  参数设置
 根据以下几个值来决定
 - tasks ：每秒的任务数，假设为500~1000
 - taskcost：每个任务花费时间，假设为0.1s
 - responsetime：系统允许容忍的最大响应时间，假设为1s
 ----
    - corePoolsize= tasks/(1/taskcost) =tasks*taskcout=50~100
    - queueCapacity =(coreSizePool/taskcost)*responsetime
    - maxPoolSize = (max(tasks)- queueCapacity)/(1/taskcost)
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

在阿里规约中，线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：Executors各个方法的弊端
1. newFixedThreadPool和newSingleThreadExecutor:
主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM。
2. newCachedThreadPool和newScheduledThreadPool:主要问题是线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，

