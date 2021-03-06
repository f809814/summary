<h2 align="center">java并发面试常见问题</h2>

##### 1. 多线程java中有几种方法可以实现一个线程？

* * *

* 继承Thread类创建线程
* 实现Runnable接口创建线程
* 实现Callable接口通过FutureTask包装器来创建Thread线程

```java
//代码来自网络
public class SomeCallable<V> extends OtherClass implements Callable<V> {

    @Override
    public V call() throws Exception {
        // TODO Auto-generated method stub
        return null;
    }
}
Callable<V> oneCallable = new SomeCallable<V>();   
//由Callable<Integer>创建一个FutureTask<Integer>对象：   
FutureTask<V> oneTask = new FutureTask<V>(oneCallable);   
//注释：FutureTask<Integer>是一个包装器，它通过接受Callable<Integer>来创建，它同时实现了Future和Runnable接口。 
  //由FutureTask<Integer>创建一个Thread对象：   
Thread oneThread = new Thread(oneTask);   
oneThread.start();   
//至此，一个线程就创建完成了。
```

##### 2. 如何停止一个正在运行的线程？

* * *
一个比较好的方法是，循环检测条件变量+interrupt()来结束线程，如下所示：

```java
public class BestPractice extends Thread {
    private volatile boolean finished = false;   // ① volatile条件变量
    public void stopMe() {
        finished = true;    // ② 发出停止信号
        interrupt();
    }
    @Override
    public void run() {
        while (!finished) {    // ③ 检测条件变量（也可以使用isInterrupted()来代替此变量）
        try {
                // do dirty work   // ④业务代码
        } catch (InterruptedException e) {
                if (finished) {
                    return;
                }
                continue;
        }
        }
    }
}
```

思考1：为什么不用stop()方法或suspend()与resume()？

stop()已被弃用，因为它会强制中断线程的执行，还会释放这个线程所持有的所有的锁对象，如果此时有其他因为请求锁而被阻塞的线程存在，则会立即获取锁，可能会造成数据的不一致。

suspend被弃用的原因是因为它会造成死锁。suspend方法和stop方法不一样，它不会破换对象和强制释放锁，相反它会一直保持对锁的占有，一直到其他的线程调用resume方法，它才能继续向下执行。假如有A，B两个线程，A线程在获得某个锁之后被suspend阻塞，这时A不能继续执行，线程B在获得相同的锁之后才能调用resume方法将A唤醒，但是此时的锁被A占有，B不能继续执行，也就不能及时的唤醒A，此时A，B两个线程都不能继续向下执行而形成了死锁。这就是suspend被弃用的原因。

思考2：volatile变量的作用？

将引用变量finished的类型前加上volatile关键字的目的是防止编译器对该变量存取时的优化，这种优化主要是缓存对变量的修改，这将使其他线程不会立刻看到修改后的finished值，从而影响退出。此外，Java标准保证被volatile修饰的变量的读写都是原子的

思考3：无法中断正在等待获取synchronized锁的线程（reentrantLock.lock()方法也是一样），可以调用reentrantLock.lockInterruptibly()方法或者带超时的tryLock方法reentrantLock.tryLock。（**TODO**）

##### 3. notify()和notifyAll()有什么区别？

* * *
锁池和等待池：

- 锁池:假设线程A已经拥有了某个对象的锁，而其它的线程想要调用这个对象的某个synchronized方法(或者synchronized块)，由于这些线程在进入对象的synchronized方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程A拥有，所以这些线程就进入了该对象的锁池中。
- 等待池:假设一个线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁后，进入到了该对象的等待池中

> Reference：[java中的锁池和等待池 ](https://link.zhihu.com/?target=http%3A//blog.csdn.net/emailed/article/details/4689220)

notify和notifyAll的区别：

-  如果线程调用了对象的 wait()方法，那么线程便会处于该对象的**等待池**中，等待池中的线程**不会去竞争该对象的锁**。
- 当有线程调用了对象的 `notifyAll`方法（唤醒所有 wait 线程）或 `notify`方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了`notify`后只要一个线程会由等待池进入锁池，而`notifyAll`会将该对象等待池内的所有线程移动到锁池中，等待锁竞争
- 优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它**还会留在锁池中**，唯有线程再次调用 `wait`()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 `synchronized` 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。

> Reference：[线程间协作：wait、notify、notifyAll ](https://link.zhihu.com/?target=http%3A//wiki.jikexueyuan.com/project/java-concurrency/collaboration-between-threads.html)

综上，所谓唤醒线程，另一种解释可以说是将线程由等待池移动到锁池，notifyAll调用后，会将全部线程由等待池移到锁池，然后参与锁的竞争，竞争成功则继续执行，如果不成功则留在锁池等待锁被释放后再次参与竞争。而notify只会唤醒一个线程。

注意：使用notify可能会出现所有线程都进入等待池，程序将挂起。

##### 4. `sleep()`和 `wait()`有什么区别?

* * *
**`sleep()`**使当前线程进入停滞状态（阻塞当前线程），让出cpu的使用、目的是不让当前线程独自霸占该进程所获的cpu资源，以留一定时间给其他线程执行的机会，`sleep`()是`Thread`类的`static`(静态)的方法，因此他不能改变对象的机锁，所以当在一个`Synchronized`块中调用`sleep()`方法是，线程虽然休眠了，但是对象的机锁并木有被释放，其他线程无法访问这个对象（即使睡着也持有对象锁），在`sleep()`休眠时间期满后，该线程不一定会立即执行，这是因为其它线程可能正在运行而且没有被调度为放弃执行，除非此线程具有更高的优先级。 

**`wait()`**方法是`Object`类里的方法；当一个线程执行到`wait()`方法时，它就进入到一个和该对象相关的等待池中，同时失去（释放）了对象的机锁（暂时失去机锁，`wait(long timeout)`超时时间到后还需要返还对象锁）,其他线程可以访问，`wait()`使用`notify`或者`notifyAlll`或者指定睡眠时间来唤醒当前等待池中的线程，**`wait`,`notify`和`notifyAll`方法必须在同步方法或同步块中调用，否则会抛出`java.lang.IllegalMonitorStateException`异常**。

> wait():This method should only be called by a thread that is the owner of this object's monitor.
>
> 例如，生产者线程向缓冲区中写入数据，消费者线程从缓冲区中读取数据。消费者线程需要等待直到生产者线程完成一次写入操作。生产者线程需要等待消费者线程完成一次读取操作。假设wait(),notify(),notifyAll()方法不需要加锁就能够被调用。此时消费者线程调用wait()正在进入状态变量的等待队列(译者注:可能还未进入)。在同一时刻，生产者线程调用notify()方法打算向消费者线程通知状态改变。那么此时消费者线程将错过这个通知并一直阻塞。因此，对象的wait(),notify(),notifyAll()方法必须在该对象的同步方法或同步代码块中被互斥地调用。

##### 5. 什么是Daemon线程？它有什么意义？

* * *
守护线程，是指用户程序在运行的时候后台提供的一种通用服务的线程，比如用于垃圾回收的垃圾回收线程。这类线程并不是用户线程不可或缺的部分，只是用于提供服务的”服务线程”。

在Java中有两类线程：User Thread(用户线程)、Daemon Thread(守护线程) ，用个比较通俗的比如，任何一个守护线程都是整个JVM中所有非守护线程的保姆：

只要当前JVM实例中尚存在任何一个非守护线程没有结束，守护线程就全部工作；只有当最后一个非守护线程结束时，守护线程随着JVM一同结束工作。
Daemon的作用是为其他线程的运行提供便利服务，守护线程最典型的应用就是 GC (垃圾回收器)，它就是一个很称职的守护者。

User和Daemon两者几乎没有区别，唯一的不同之处就在于虚拟机的离开：如果 User Thread已经全部退出运行了，只剩下Daemon Thread存在了，虚拟机也就退出了。 因为没有了被守护者，Daemon也就没有工作可做了，也就没有继续运行程序的必要了。

##### 6. java如何实现多线程之间的通讯和协作？

* * *
> 引用博客--[Java 并发：线程间通信与协作](https://blog.csdn.net/justloveyou_/article/details/54929949)

##### 7. 锁，什么是可重入锁（ReentrantLock）？

* * *


##### 8. 当一个线程进入某个对象的一个synchronized的实例方法后，其它线程是否可进入此对象的其它方法？

* * *
要依据其他方法是否也为synchronized的实例方法：

- 如果没有`synchronized`修饰，可以访问；
- 如果有`synchronized`修饰但是为`static`方法，可以访问；
- 如果是`synchronized`修饰的的实例方法，不能访问；
- 如果这个方法内部调用了`wait`，并进入了等待状态(已经释放了锁)，则可以进入其他`synchronized`方法。
  - 注意:调用的`wait`方法必须为**`this.wait()`** 才能访问其他`synchronized`方法，因为只有`this.wait()`才是释放的本对象的锁，其他线程才能访问本对象的其他同步示例方法。

##### 9. synchronized和java.util.concurrent.locks.Lock的异同？

* * *
##### 10. 乐观锁和悲观锁的理解及如何实现，有哪些实现方式？
* * *
##### 11. 并发框架SynchronizedMap和ConcurrentHashMap有什么区别？
* * *
##### 12. CopyOnWriteArrayList可以用于什么应用场景？
* * *
##### 13. 线程安全什么叫线程安全？servlet是线程安全吗?
* * *
一个不论运行时（Runtime）如何调度线程都不需要调用方提供额外的同步和协调机制还能正确地运行的类是线程安全的

Servlet本身是无状态的，**一个无状态的Servlet是绝对线程安全的，无状态对象设计也是解决线程安全问题的一种有效手段**。所以，servlet是否线程安全是由它的实现来决定的，如果它内部的属性或方法会被多个线程改变，它就是线程不安全的，反之，就是线程安全的

- 避免使用实例变量
- 避免使用非线程安全的集合
- 在多个Servlet中对某个外部对象(例如文件)的修改是务必加锁（Synchronized，或者ReentrantLock），互斥访问。
- 属性的线程安全：ServletContext、HttpSession是非线程安全的；ServletRequest是线程安全的。
  - HttpSession只能在处理属于同一个Session的请求的线程中被访问，因此Session对象的属性访问理论上是线程安全的，但当用户打开多个同属于一个进程的浏览器窗口，在这些窗口的访问属于同一个Session，会出现多次请求，需要多个工作线程来处理请求，可能造成同时多线程读写属性。

##### 14. 同步有几种实现方法？

* * *
##### 15. volatile有什么用？能否用一句话说明下volatile的应用场景？
* * *
Java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量。Java语言提供了volatile，在某些情况下比锁要更加方便。如果一个字段被声明成volatile，Java线程内存模型确保所有线程看到这个变量的值是一致的。 它在多处理器开发中保证了共享变量的“可见性”。可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。

***volatile的内存语义：***

- 可见性：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入；
- 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。 

***volatile写和读的内存语义：***

当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存 ；当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量 。

- 线程A写一个volatile变量，实质上是线程A向接下来将要读这个volatile变量的某个线程发出了（其对共享变量所做修改的）消息。

- 线程B读一个volatile变量，实质上是线程B接收了之前某个线程发出的（在写这个volatile变量之前对共享变量所做修改的）消息。

- 线程A写一个volatile变量，随后线程B读这个volatile变量，这个过程实质上是线程A通过主内存向线程B发送消息。 

***volatile内存语义的实现：***

规定了内存重排序规则，具体见

> Java并发编程的艺术--3.4.4节

***锁的内存语义：***

当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中 ；当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使得被监视器保护的临界区代码必须从主内存中读取共享变量 。

- 线程A释放一个锁，实质上是线程A向接下来将要获取这个锁的某个线程发出了（线程A对共享变量所做修改的）消息 ；

- 线程B获取一个锁，实质上是线程B接收了之前某个线程发出的（在释放这个锁之前对共享变量所做修改的）消息 ；

- 线程A释放锁，随后线程B获取这个锁，这个过程实质上是线程A通过主内存向线程B发送消息 。

##### 16. 请说明下java的内存模型及其工作流程。

* * *


##### 17. 为什么代码会重排序？

* * *

重排序是指编译器和处理器**为了优化程序性能**而对指令序列进行重新排序的一种手段 。编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。 这里所说的数据依赖性**仅针对单个处理器中执行的指令序列和单个线程中执行的操作**，不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑。 

在计算机中，软件技术和硬件技术有一个共同的目标：在不改变程序执行结果的前提下，尽可能提高并行度。编译器和处理器遵从这一目标，从happens-before的定义我们可以看出，JMM同样遵从这一目标。 