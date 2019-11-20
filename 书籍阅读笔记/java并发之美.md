### 基础
 - wait
   -  notify()或者notifyAll()
   -  其他线程调用了该线程的 interrupt()方法
   - 如果调用 wait()方法的线程没有事先获取该对象的监视器锁，则 调用 wait()方法时调用线程会抛出 IllegalMonitorStateException异常
 - notify()
   - 唤醒一个共享变量随机线程
   - 并不是立刻执行，还需要竞争
   - 只有得到共享变量的监视器锁后
 - notifyAll()
   - 唤醒所有在该共享变量上由于调用 wait 系列方法而被挂起的线程。
 - sleep()
   - 锁不让出
 - yield()
   - 当线程调用 sleep 方法时调用线程 会被阻塞挂 起指定的时间，在这期间线程调度器不会去调度该线程 。 而调用 yield 方法时，线程只是 让出自己剩余的时间片，并没有被阻塞挂起，而是处于就绪状态，线程调度器下 一次调度 时就有可能调度到当前线程执行 
- 线程上下文切换
  - 当前线程的 CPU 时 间片 使用完处于就绪状态 时，当前线程 被其他线程中断时 。
- synchronized
  - 使用synchronized阻塞一个线程时，需要从用户态切换到内核态执行阻塞操作（上下文切换）。
  - 线程重新调度开销
  - 可见性
     - 直接从主内存获取，退出时，刷新到主内存
- volatile
  - 只保证可见性，有序性
  - 写入变量值不依赖变量的当前值
- 指令重排序
  - 对不存在数据依赖性的指令重排序
#### CopyOnWirteArrayList
- 读写分离思想。写时复制，增删改都使用独占锁，保证某个时间只有一个线程对list进行修改
- 迭代器遍历的数组是一个快照
- CopyOnWriteArraySet底层就是一个CopyOnWirteArrayList
- 底层数组使用volatile修饰
- 适用于多线程环境中，读多写少的场景
#### 并发包的原子操作类
- AtomicLong（AtomicBoolean,AtomicInteger）
  - 高并发下大量线程会同时去竞争更新同一个原子变量，但是由于同时只有一个线程的cas操作成功，造成大量线程竞争失败后，会通过无限循环不断进行自旋尝试cas的操作，造成cpu浪费
- LongAdder
### AQS
- 公平与非公平靠hasQueuedPredecessors函数保证。看当前节点的前驱节点是否也在等待获取资源，如果是自己则放弃。放入到AQS阻塞队列中
- 共享与独占：共享锁的主要特征表现在当一个等待队列的共享节点成功获取到锁以后，就去唤醒后面的节点。
  - setHeadAndPropagate方法
  - 独占锁只唤醒接下来的一个node
#### countDownLatch
- 场景：主线程等待子线程执行完毕后进行汇总
- 一次性
- 优于join
   - join线程池无法使用
   - join需要子线程执行完毕后
#### cyclicBarrier
- 场景：一组线程到达一个状态后再全部执行
- 基于独占锁.使用lock的条件变量trip的await与signal进行同步
- 可以重用
  - 内部2个变量。count与parties。初始化.parties=count。每次await(),count-1.当count=0.parties赋值给count。从而复用
- 核心方法dowait
#### Semaphore
- 内部计数器递增，一开始并不关心需要同步的线程个数，等到调用aquire方法时再制定你需要同步的个数
#### ReentrantReadWriteLock
- 读写分离策略，允许多个线程可以同时获取读锁
#### ThreadPoolExecutor
- ctl 32位int
  - 高3位线程池状态
    - RUNNING：接受新任务并处理阻塞队列的任务
    - SHUTDOWN：拒绝新任务但是处理阻塞队列里的任务
    - STOP：拒绝新任务并且抛弃阻塞队列的任务，同时会中断正在处理的任务
    - TIDYING：所有任务都执行完吼当前线程池活动线程数为0，将调用terminated方法
    - TERMINATED:终止状态，terminated方法调用完成以后的状态 
  - 低29位线程池个数
- 状态之间转换
 - RUNNING->SHUTDOWN:显示调用shutdown()
 - RUNNIN/SHUTDOWN->STOP:显示调用shutdownNow()
 - SHUTDOWN->TIDYING:线程池和任务队列都为空
 - STOP->TIDYING:线程池为空
 - TIDYING->TERMINATED:当terminated()hook方法执行完成时
- 整体流程：（execute方法）
  - 如果当前线程池个数小于coreSize,则新开启线程
  - 如果大于coreSize,则添加到阻塞队列中
    - 状态为RUNNING
    - 二次检查
  - 队列如果满，则新建线程(<maxPoolSize)
  - 如果大于maxPoolSize,则执行拒绝策略
#### 并发队列
- LinkedBlockingQueue
  - 生产者-消费者模型
  - 单向链表，头尾节点进行入队，出队。使用独占锁。对头尾节点的独占锁都配备了一个条件队列，用来存放被阻塞的线程
  ![](https://i.loli.net/2019/10/21/zNC7cZPbr6QFHi4.png)
#### java内存模型
- 内存模型
  - CPu和缓存一致性
    - 出现原因：内存读写速度越来越跟不上cpu计算速度
    - 出现Cpu缓存
      - 一级缓存。二级缓存。三级缓存。
      - 多核多cpu情况下就出线缓存一致性问题
  - 处理器优化和指令重排
    - 为了使处理器内部的运算单元能够充分被利用，处理器会对输入代码进行乱序执行处理
缓存一致性就是可见性。处理器优化就是原子性。指令重排就是有序性
内存模型定义了共享内存系统中多线程读写规范
- 两种方式
  - 限制处理器优化
  - 内存屏障
- java内存模型就是一种符合内存模型规范的，屏蔽了各种硬件和操作系统的访问差异。保证java程序在各种平台下对内存的访问都能保证效果一致的机制和规范
![](https://i.loli.net/2019/11/11/nWEMVxGDAgj4UXT.png)