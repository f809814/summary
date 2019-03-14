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

##### 3. notify()和notifyAll()有什么区别？

* * *
##### 4. sleep()和 wait()有什么区别?
* * *
##### 5. 什么是Daemon线程？它有什么意义？
* * *
##### 6. java如何实现多线程之间的通讯和协作？
* * *
##### 7. 锁什么是可重入锁（ReentrantLock）？
* * *
##### 8. 当一个线程进入某个对象的一个synchronized的实例方法后，其它线程是否可进入此对象的其它方法？
* * *
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
##### 14. 同步有几种实现方法？
* * *
##### 15. volatile有什么用？能否用一句话说明下volatile的应用场景？
* * *
##### 16. 请说明下java的内存模型及其工作流程。
* * *
##### 17. 为什么代码会重排序？
* * *