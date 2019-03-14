##### 1. 多线程java中有几种方法可以实现一个线程？

* * *

* 继承Thread类创建线程

* 实现Runnable接口创建线程

* 实现Callable接口通过FutureTask包装器来创建Thread线程

  ```java
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