##AQS
一句话概括：多线程访问共享资源的同步器框架。
![](https://i.loli.net/2019/01/16/5c3f1f5f4f515.png)
#### 原理

![](https://i.loli.net/2019/01/16/5c3f20e26f55f.png)
**如果被请求的资源空闲，则当前请求资源的线程设置为有效的工作线程，并且将共享资源设定为锁定状态。如果被占用，就需要一套线程阻塞，等待的机制，这个机制是使用队列锁实现，即暂时获取不到锁的线程被加入队列中**
### 核心属性以及方法
```
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```
### 资源共享方式

- Exclusive（独占）：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：
  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- Share（共享）：多个线程可同时执行

不同的自定义同步器争用共享资源的方式也不同。

自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可（一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。），

至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在上层已经帮我们实现好了。

#### 常见子类
- ReentrantLock
- Semaphore
- ReentrantReadWriteLock
- SynchronousQueue
- CountDownLatch
- CyclicBarrier
#### CountDownLatch
CountDownLatch是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程的操作执行完后再执行

##### 原理：
CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务
##### 主要方法
```
#count为闭锁需要等到的线程数量
public void CountDownLatch(int count) {...}
# 调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public void await() throws InterruptedException { };  
# 和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { }; 
# 将count值减1
public void countDown() { }; 
```
#### CyclicBarrier

用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行
##### 主要方法
```
#参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。
public CyclicBarrier(int parties, Runnable barrierAction) {
}
 
public CyclicBarrier(int parties) {
}
public int await() throws InterruptedException, BrokenBarrierException { };
public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };
```

#### 二者区别
CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：

CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；

而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；

另外，CountDownLatch是不能够重用的，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用。
#### Semaphore
Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数
