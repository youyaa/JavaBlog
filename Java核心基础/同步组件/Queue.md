## 单端非阻塞队列和阻塞队列

### 非阻塞队列

ConcurrentLinkedQueue

ConcurrentLinkedQueue是一个基于链接节点的无边界的线程安全队列，遵循队列的FIFO原则，队尾入队，队首出队。采用CAS算法来实现的。

>1. ConcurrentLinkedQueue的.size() 是要遍历一遍集合的，很慢的，所以尽量要避免用size()方法。
>2. 使用了这个ConcurrentLinkedQueue 类之后还是需要自己进行同步或加锁操作。例如queue.isEmpty()后再进行队列操作queue.add()是不能保证安全的，因为可能queue.isEmpty()执行完成后，别的线程开始操作队列。

 

### 阻塞队列 

BlockingQueue

不支持Null值，会报错。

BlockingQueue 对插入操作、移除操作、获取元素操作提供了四种不同的方法用于不同的场景中使用：

1. 抛出异常
2. 返回特殊值（null 或 true/false，取决于具体的操作）
3. 阻塞等待此操作，直到这个操作成功
4. 阻塞等待此操作，直到成功或者超时指定时间。总结如下：

| 操作 | 抛出异常  | 特殊值   | 阻塞   | 超时                 |
| :--: | :-------- | :------- | :----- | :------------------- |
| 插入 | add(e)    | offer(e) | put(e) | offer(e, time, unit) |
| 移除 | remove()  | poll()   | take() | poll(time, unit)     |
| 检查 | element() | peek()   | 不可用 | 不可用               |

#### ArrayBlockingQueue

ArrayBlockingQueue是一个由数组实现的有界阻塞队列。该队列采用FIFO的原则对元素进行排序添加的。

ArrayBlockingQueue为有界且固定，其大小在构造时由构造函数来决定，确认之后就不能再改变了。

ArrayBlockingQueue支持对等待的生产者线程和使用者线程进行排序的可选公平策略，但是在默认情况下不保证线程公平的访问，在构造时可以选择公平策略（fair = true）。公平性通常会降低吞吐量，但是减少了可变性和避免了“不平衡性”。

ArrayBlockingQueue内部使用可重入锁ReentrantLock + Condition来完成多线程环境的并发操作。

- items，一个定长数组，维护ArrayBlockingQueue的元素
- takeIndex，int，为ArrayBlockingQueue队首位置
- putIndex，int，ArrayBlockingQueue队尾位置
- count，元素个数
- lock，锁，ArrayBlockingQueue出列入列都必须获取该锁，两个步骤公用一个锁。

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, Serializable {
    private static final long serialVersionUID = -817911632652898426L;
    final Object[] items;
    int takeIndex;
    int putIndex;
    int count;
    // 重入锁
    final ReentrantLock lock;
    // notEmpty condition
    private final Condition notEmpty;
    // notFull condition
    private final Condition notFull;
    transient ArrayBlockingQueue.Itrs itrs;
}
```

#### LinkedBlockingQueue

LinkedBlockingQueue和ArrayBlockingQueue的使用方式基本一样，但还是有一定的区别：

1. **队列的数据结构不同**

   ArrayBlockingQueue是一个由数组支持的有界阻塞队列

   LinkedBlockingQueue是一个基于链表的有界（可设置）阻塞队列 

2. **队列中锁的实现不同**

ArrayBlockingQueue实现的队列中的锁是没有分离的，即生产和消费用的是同一个锁；

LinkedBlockingQueue实现的队列中的锁是分离的，即生产用的是putLock，消费是takeLock

3. **在生产或消费时操作不同**

ArrayBlockingQueue实现的队列中在生产和消费的时候，是直接将枚举对象插入或移除的；

LinkedBlockingQueue实现的队列中在生产和消费的时候，需要把枚举对象转换为Node进行插入或移除，会影响性能 

4. **队列大小初始化方式不同**

ArrayBlockingQueue实现的队列中必须指定队列的大小；

LinkedBlockingQueue实现的队列中可以不指定队列的大小，但是默认是Integer.MAX_VALUE

#### PriorityBlockingQueue

类似于ArrayBlockingQueue内部使用一个独占锁来控制，同时只有一个线程可以进行入队和出队。

PriorityBlockingQueue是一个优先级队列，它在java.util.PriorityQueue的基础上提供了可阻塞的读取操作。它是无界的，就是说向Queue里面增加元素没有数量限制，但可能会导致内存溢出而失败。

PriorityBlockingQueue始终保证出队的元素是优先级最高的元素，并且可以定制优先级的规则，内部使用二叉堆，通过使用一个二叉树最小堆算法来维护内部数组，这个数组是可扩容的，当当前元素个数>=最大容量时候会通过算法扩容。值得注意的是为了避免在扩容操作时候其他线程不能进行出队操作，实现上使用了先释放锁，然后通过CAS保证同时只有一个线程可以扩容成功。

>1、优先队列不允许空值，而且不支持non-comparable（不可比较）的对象，比如用户自定义的类。优先队列要求使用Java Comparable和Comparator接口给对象排序，并且在排序时会按照优先级处理其中的元素。自定义的类若不实现comparable接口，在运行时会报错“cannot be cast to java.lang.Comparable”
>
>2、优先队列的头是基于自然排序或者Comparator排序的最小元素。如果有多个对象拥有同样的排序，那么就可能随机地取其中任意一个。也可以通过提供的Comparator（比较器）在队列实现自定的排序。当我们获取队列时，返回队列的头对象。
>
>3、优先队列的大小是不受限制的，但在创建时可以指定初始大小，当我们向优先队列增加元素的时候，队列大小会自动增加。
>
>4、PriorityQueue是非线程安全的，所以Java提供了PriorityBlockingQueue（实现BlockingQueue接口）用于Java多线程环境。

#### SynchronousQueue 

SynchronousQueue，实际上它不是一个真正的队列，因为它不会为队列中元素维护存储空间。与其他队列不同的是，它维护一组线程，这些线程在等待着把元素加入或移出队列。SynchronousQueue没有存储功能，因此put和take会一直阻塞，直到有另一个线程已经准备好参与到交付过程中。

仅当有足够多的消费者，并且总是有一个消费者准备好获取交付的工作时，才适合使用同步队列。这种实现队列的方式看似很奇怪，但由于可以直接交付工作，从而降低了将数据从生产者移动到消费者的延迟。

直接交付方式还会将更多关于任务状态的信息反馈给生产者。当交付被接受时，它就知道消费者已经得到了任务，而不是简单地把任务放入一个队列——这种区别就好比将文件直接交给同事，还是将文件放到她的邮箱中并希望她能尽快拿到文件。

SynchronousQueue对于正在等待的生产者和使用者线程而言，默认是非公平排序，也可以选择公平排序策略。但是，使用公平所构造的队列可保证线程以 FIFO 的顺序进行访问。 公平通常会降低吞吐量，但是可以减小可变性并避免得不到服务。

**SynchronousQueue特点：**

- 是一种阻塞队列，其中每个 put 必须等待一个 take，反之亦然。同步队列没有任何内部容量，甚至连一个队列的容量都没有。
- 是线程安全的，是阻塞的。
- 不允许使用 null 元素。
- 公平排序策略是指调用put的线程之间，或take的线程之间的线程以 FIFO 的顺序进行访问。
- SynchronousQueue的方法：
  - iterator()： 永远返回空，因为里面没东西。
  - peek() ：永远返回null。
  - put() ：往queue放进去一个element以后就一直wait直到有其他thread进来把这个element取走。
  - offer() ：往queue里放一个element后立即返回，如果碰巧这个element被另一个thread取走了，offer方法返回true，认为offer成功；否则返回false。
  - offer(2000, TimeUnit.SECONDS) ：往queue里放一个element但等待时间后才返回，和offer()方法一样。
  - take() ：取出并且remove掉queue里的element，取不到东西他会一直等。
  - poll() ：取出并且remove掉queue里的element，方法立即能取到东西返回。否则立即返回null。
  - poll(2000, TimeUnit.SECONDS) ：等待时间后再取，并且remove掉queue里的element,
  - isEmpty()：永远是true。
  - remainingCapacity() ：永远是0。
  - remove()和removeAll() ：永远是false。

#### LinkedTransferQueue

#### DelayQueue



## 双端非阻塞和阻塞队列

非阻塞队列：ConcurrentLinkedDeque

阻塞队列：LinkedBlockingDeque

