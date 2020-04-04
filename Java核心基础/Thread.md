## 何为进程，何为线程

**进程：**

是指内存中运行的一个应用程序，每个进程都有一个独立的内存空间，一个应用程序可以同时运行多个进程；进程也是程序的一次执行过程，是系统运行程序的基本单位；系统运行一个程序即是一个进程从创建、运行到消亡的过程。

**线程：**

进程内部的一个独立执行单元；一个进程可以同时并发的运行多个线程，所以线程也被称为轻量级进程。线程是CPU调度的最小单位，线程的切换可能会引起进程的切换。

> Hotspot JVM 中的 Java 线程与原生操作系统线程有直接的映射关系。当线程本地存储、缓冲区分配、同步对象、栈、程序计数器等准备好以后，就会创建一个操作系统原生线程。 Java 线程结束，原生线程随之被回收。操作系统负责调度所有线程，并把它们分配到任何可用的 CPU 上。当原生线程初始化完毕，就会调用 Java 线程的 run() 方法。当线程结束时，会释放原生线程和 Java 线程的所有资源。

## 线程状态和切换

操作系统定义的线程状态：

1. 新建  New
2. 就绪  Ready                    **表示线程已经被创建，正在等待CPU调度。**
3. 运行  Running                **表示线程获得了CPU使用权，正在进行运算**
4. 阻塞  Blocked                 **表示线程阻塞，让出CPU的使用权**
5. 消亡  Terminated

JAVA中定义的线程状态：

查看Thread类的源码：

```java
public enum State {
        NEW,
        RUNNABLE,         
        BLOCKED,          
        WAITING,					
        TIMED_WAITING,
        TERMINATED;
    }
```

1. NEW ：             新建。
2. RUNNABLE：  可运行。可能正在运行，也可能在等待CPU调度。
3. BLOCKED：     锁阻塞。当线程等待 synchronized 的隐式锁。synchronized 修饰的方法、代码块同一时刻只允许一个线程执行，其他线程只能等待，这种情况下，等待的线程就会从 RUNNABLE 转换到 BLOCKED 状态。而当等待的线程获得 synchronized 隐式锁时，就又会从 BLOCKED 转换到 RUNNABLE 状态。
4. WAITING：      无限等待。一个线程在等待另一个线程执行一个（唤醒）动作时，该线程进入Waiting状态。进入这个状态后是不能自动唤醒的，必须等待另一个线程调用notify或者notifyAll方法才能够唤醒。调用wait()，join()会进入这个状态。
5. TIMED_WAITING： 计时等待。 同waiting状态，有几个方法有超时参数，调用他们将进入Timed Waiting状态。这一状态将一直保持到超时期满或者接收到唤醒通知。带有超时参数的常用方法有Thread.sleep 、Object.wait。
6. TERMINATED： 终止。线程正常执行完run()方法，或者interrupt()。