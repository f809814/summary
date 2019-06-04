<h4 align="center">Java并发机制原理、锁、并发容器、工具类和框架</h4>

##### 1. Java中各种锁的分析

***

> [不可不说的Java“锁”事]: https://tech.meituan.com/2018/11/15/java-lock.html	"来自美团技术团队"

ReentrantReadWriteLock，JDK提供的一种读写锁，内部有两个锁：ReadLock和WriteLock，一个读锁一个写锁，合称“读写锁”，*读锁是共享锁，写锁是独享锁。读锁的共享锁可保证并发读非常高效，而读写、写读、写写的过程互斥，因为读锁和写锁是分离的*。以下为读锁的部分源码：

```java
protected final int tryAcquireShared(int unused) {
            /*
             * Walkthrough:
             * 1. If write lock held by another thread, fail.
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */
            Thread current = Thread.currentThread();
            int c = getState();
    		//有其他线程持有写锁，获取失败
            if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current)
                return -1;
            int r = sharedCount(c);
    		//本线程持有写锁也可以再获取读锁，因为同一线程内读写不存在不同步的问题
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    firstReaderHoldCount++;
                } else {
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            return fullTryAcquireShared(current);//待分析
        }
```

有其他线程持有写锁，获取失败，但是如果是本线程持有写锁，则可以获取读锁，同一线程内不存在不同步的问题 。
