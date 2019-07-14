<h2 align="center">Java线程生命周期</h2>
#### 1.Java线程的6种状态


- 初始(NEW)：新创建了一个线程对象，但还没有调用start()方法。
- 运行(RUNNABLE)：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。
  线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。
- 阻塞(BLOCKED)：表示线程阻塞于锁。
- 等待(WAITING)：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
- 超时等待(TIMED_WAITING)：该状态不同于WAITING，它可以在指定的时间后自行返回。
- 终止(TERMINATED)：表示该线程已经执行完毕。

**(1) RUNNABLE 与 BLOCKED 的状态转换**

只有一种场景会触发这种转换，就是**线程等待 synchronized 的隐式锁**。

在线程调用阻塞式 API 时，是否会转换到 BLOCKED 状态呢？在操作系统层面，线程是会转换到休眠状态的，但是在 JVM 层面，Java 线程的状态不会发生变化，也就是说 Java 线程的状态会依然保持 RUNNABLE 状态。JVM 层面并不关心操作系统调度相关的状态，因为在 JVM 看来，等待 CPU 使用权（操作系统层面此时处于可执行状态）与等待 I/O（操作系统层面此时处于休眠状态）没有区别，都是在等待某个资源，所以都归入了 RUNNABLE 状态。而我们平时所谓的 Java 在调用阻塞式 API 时，线程会阻塞，指的是操作系统线程的状态，并不是 Java 线程的状态。

**(2) RUNNABLE 与 WAITING 的状态转换**

有三种场景会触发这种转换。

- 第一种场景，**获得 synchronized 隐式锁的线程，调用无参数的 Object.wait() 方法**。其中，wait() 方法我们在上一篇讲解管程的时候已经深入介绍过了，这里就不再赘述。
- 第二种场景，**调用无参数的 Thread.join() 方法**。其中的 join() 是一种线程同步方法，例如有一个线程对象 thread A，当调用 A.join() 的时候，执行这条语句的线程会等待 thread A 执行完，而等待中的这个线程，其状态会从 RUNNABLE 转换到 WAITING。当线程 thread A 执行完，原来等待它的线程又会从 WAITING 状态转换到 RUNNABLE。
- 第三种场景，**调用 LockSupport.park() 方法**。其中的 LockSupport 对象， Java 并发包中的锁，都是基于它实现的。调用 LockSupport.park() 方法，当前线程会阻塞，线程的状态会从 RUNNABLE 转换到 WAITING。调用 LockSupport.unpark(Thread thread) 可唤醒目标线程，目标线程的状态又会从 WAITING 状态转换到 RUNNABLE。

**(3) RUNNABLE 与 TIMED_WAITING 的状态转换**

- 有五种场景会触发这种转换：
- 调用带超时参数的 **Thread.sleep(long millis)** 方法；
- 获得 synchronized 隐式锁的线程，调用带超时参数的 Object.wait(long timeout) 方法；
- 调用带超时参数的 Thread.join(long millis) 方法；
- 调用带超时参数的 LockSupport.parkNanos(Object blocker, long deadline) 方法；
- 调用带超时参数的 LockSupport.parkUntil(long deadline) 方法。

**(4) RUNNABLE 与 TIMED_WAITING 的状态转换**

Java 刚创建出来的 Thread 对象就是 NEW 状态，从 NEW 状态转换到 RUNNABLE 状态很简单，只要调用线程对象的 start() 方法就可以了。

**(5) RUNNABLE 与 TIMED_WAITING 的状态转换**

线程执行完 run() 方法后，会自动转换到 TERMINATED 状态，如果执行 run() 方法的时候异常抛出，也会导致线程终止。如果需要强制终止线程，stop方法已不再推荐使用，因为会真的杀死线程，如果线程还持有未释放的synchronized锁，那么其他线程就没有机会获得这个锁了，推荐采用**interrupt()** 方法，分为两种情况：

- 线程被动通过异常接收到通知
  - 当处于 WAITING、TIMED_WAITING 状态时，其他线程如果调用了interrupt() 方法，会使线程  返回到 RUNNABLE 状态，同时线程 A 的代码会触发 InterruptedException 异常；
  - 当线程 A 处于 RUNNABLE 状态时，并且阻塞在 java.nio.channels.InterruptibleChannel 上时，如果其他线程调用线程 A 的 interrupt() 方法，线程 A 会触发 java.nio.channels.ClosedByInterruptException 这个异常；而阻塞在 java.nio.channels.Selector 上时，如果其他线程调用线程 A 的 interrupt() 方法，线程 A 的 java.nio.channels.Selector 会立即返回。

- 线程主动检测中断信号

线程 A主动调用 isInterrupted() 方法来检测中断状态

---

#### 2.创建多少个线程是合适的

对于 CPU 密集型的计算场景，理论上“线程的数量 =CPU 核数”就是最合适的，实际中可以设为“CPU 核数 +1“，这样的话，当线程因为偶尔的内存页失效或其他原因导致阻塞时，这个额外的线程可以顶上，从而保证 CPU 的利用率。

对于 I/O 密集型的计算场景，最佳线程数 =CPU 核数 * [ 1 +（I/O 耗时 / CPU 耗时）]，可以看做一个线程执行cpu计算任务，其他k个线程执行I/O任务，其中k =  I/O 耗时 / CPU 耗时。

