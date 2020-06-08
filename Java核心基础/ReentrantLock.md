## 为什么要有ReentrantLock

在如何避免死锁中有一个方法：破坏不可抢占条件，但是Synchronized在获取锁失败后，就会进入阻塞状态。

我们希望是：

>  对于“不可抢占”这个条件，占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源，这样不可抢占这个条件就破坏掉了。

三种方案：

1. **能够响应中断。**synchronized 的问题是，持有锁 A 后，如果尝试获取锁 B 失败，那么线程就进入阻塞状态，一旦发生死锁，就没有任何机会来唤醒阻塞的线程。但如果阻塞状态的线程能够响应中断信号，也就是说当我们给阻塞的线程发送中断信号的时候，能够唤醒它，那它就有机会释放曾经持有的锁 A。这样就破坏了不可抢占条件了。
2. **支持超时。**如果线程在一段时间之内没有获取到锁，不是进入阻塞状态，而是返回一个错误，那这个线程也有机会释放曾经持有的锁。这样也能破坏不可抢占条件。
3. **非阻塞地获取锁。**如果尝试获取锁失败，并不进入阻塞状态，而是直接返回，那这个线程也有机会释放曾经持有的锁。这样也能破坏不可抢占条件。

体现在ReentrantLock中就是：

```java 
// 支持中断的API
void lockInterruptibly() 
  throws InterruptedException;
// 支持超时的API
boolean tryLock(long time, TimeUnit unit) 
  throws InterruptedException;
// 支持非阻塞获取锁的API
boolean tryLock();
```

## ReentrantLock如何保证可见性

```java
class X {
  private final Lock rtl =
  new ReentrantLock();
  int value;
  public void addOne() {
    // 获取锁
    rtl.lock();  
    try {
      value+=1;
    } finally {
      // 保证锁能释放
      rtl.unlock();
    }
  }
}
```

它是利用了 volatile 相关的 Happens-Before 规则。Java SDK 里面的 ReentrantLock，内部持有一个 volatile 的成员变量 state，获取锁的时候，会读写 state 的值；解锁的时候，也会读写 state 的值（简化后的代码如下面所示）。也就是说，在执行 value+=1 之前，程序先读写了一次 volatile 变量 state，在执行 value+=1 之后，又读写了一次 volatile 变量 state。根据相关的 Happens-Before 规则：

1. 顺序性规则：对于线程 T1，value+=1 Happens-Before 释放锁的操作 unlock()；
2. volatile 变量规则：由于 state = 1 会先读取 state，所以线程 T1 的 unlock() 操作 Happens-Before 线程 T2 的 lock() 操作；
3. 传递性规则：线程 T1 的 value+=1 Happens-Before 线程 T2 的 lock() 操作。



## ReentrantLock实现原理

基于AQS的同步组件。

## 结构

继承自AQS的Sync静态内部类，而有继承自Sync的两个NonfairSync和FairSync静态内部类，表示非公平锁和公平锁。

## 加锁过程

1、设置AbstractQueuedSynchronizer的state为1。

2、设置AbstractOwnableSynchronizer的thread为当前线程。

3、如果获取锁失败，则进入到CLH队列中，并再次判断能都获取到锁，否则调用LockSupport.park(this);阻塞当前队列。

### 解锁过程

1、获取到state的值，并不断-1。

2、设置AbstractOwnableSynchronizer的thread为null。

3、获取CLH队列上的第一个节点，调用LockSupport.unpark(s.thread);唤醒。

### 公平锁的原理

公平锁和非公平锁锁的却别就在于，加锁的时候，公平锁这样一行代码：

```java
public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```

1. 此时CLH中没有其线程等待，加锁成功。
2. 该线程就为CLH中的第一个节点，加锁成功。
3. 其他情况，加锁失败，进入阻塞队列等待。
