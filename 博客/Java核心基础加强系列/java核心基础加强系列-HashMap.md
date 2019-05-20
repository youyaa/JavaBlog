> HashMap作为我们日常使用的最频繁的一类容器，我们有必要了解其内部的实现原理。除此之外，Hash Map也极受面试官的喜爱，几乎成了Java面试必问。


## 什么是HashMap
HashMap是基于hash算法的一种key-value键值对形式的非同步容器，允许插入key和value为null的数据。

## HashMap的底层分析
Hash算法是实现HashMap的关键算法。


### 三个非常关键的点

equals：是否同一个对象实例。注意，是“实例”。比如String s = new String("test");  s.equals(s), 这就是同一个对象实例的比较；
在Object类的equals方法是和(==)一样的:
```java
 public boolean equals(Object obj) {
        return (this == obj);
    }
```
但是实现可以对该方法进行重写,以实现自己所需的功能：例如String类就对该方法进行了重写，用于判断两个字符串是否相等。

等号(==)：对比对象实例的内存地址（也即对象实例的ID），来判断是否是同一对象实例；又可以说是判断对象实例是否物理相等；

Hashcode：我觉得可以这样理解：并不是对象的内存地址，而是利用hash算法，对对象实例的一种描述符（或者说对象存储位置的hash算法映射）——对象实例的哈希码。



## HashMap常见的面试题

参考文章：https://juejin.im/post/5c1da988f265da6143130ccc

- HashMap的工作原理

HashMap基于hashing原理，我们通过put()和get()方法储存和获取对象。当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计算hashcode，返回的hashCode用于找到bucket位置来储存Entry对象。如果该位置已经有元素了,调用equals方法判断是否相等,相等的话就进行替换值,不相等的话,放在链表里面.

当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。 HashMap在每个链表节点中储存键值对对象。


![](https://img-blog.csdn.net/20180407081004596)

- 平时在使用HashMap时一般使用什么类型的元素作为Key，如何自定义key的类型？

面试者通常会回答，使用String或者Integer这样的类。这个时候可以继续追问为什么使用String、Integer呢？这些类有什么特点？如果面试者有很好的思考，可以回答出这些类是Immutable(不可变)的，并且这些类已经很规范的覆写了hashCode()以及equals()方法。作为不可变类天生是线程安全的，而且可以很好的优化比如可以缓存hash值，避免重复计算等等，那么基本上这道题算是过关了。

>重载hashCode()是为了对同一个key，能得到相同的HashCode，这样HashMap就可以定位到我们指定的桶上。

>重载equals()是为了向HashMap表明当前对象和key上所保存的对象是相等的，这样我们才真正地获得了这个key所对应的这个键值对。


- 如何衡量一个hash算法的好坏，你知道的常用hash算法有哪些？

如果面试者的技术面比较宽，或者算法基础以及数论基础比较好，这个问题才可以做很好的回答。首先，hashCode()不要求唯一但是要尽可能的均匀分布，而且算法效率要尽可能的快。如果面试者能回答出一些常用的算法，比如MurMurHash（萌萌哒的名字）基本上已经可以俘获面试官了。如果面试者有编译器的背景说出了如何在编译领域使用完美哈希的场景，那就太棒了，毕竟编译原理学的好的人太少了。当然不要忘记了，还可以再考察一下java中String类的hashCode()的实现：

为什么 h = 31 * h + val[off++]; 这一行使用31 ，而不是别的数字，这是一个魔术吗？

如果都结束了，不要忘了再问一句你知道hash攻击吗？有避免手段吗？就看面试者对各个jdk版本对HashMap的优化是否了解了。这就引出了另一个数据结构红黑树了。可以根据岗位需要继续考察rb-tree，b-tree，lsm-tree等常用数据结构以及典型应用场景。


- HashMap不是线程安全的，那么在并发下会有什么问题呢？

其实这已经开始考察面试者对并发知识的掌握情况了。HashMap在resize时候如果多个线程并发操作如何导致死循环的(resize时形成了环形链表)。面试者不一定知道，但是可以让面试者分析。毕竟很多类库在并发场景中不恰当使用HashMap导致过生产问题。


- ConcurrentHashMap vs HashTable 他们是如何处理并发的？为什么有了ConcurrentHashMap 没有把 HashTable 用@Deprecated注解废弃掉？

这个时候问题已经升级了，希望面试者分析过这两个类的源代码。我们是希望面试者能够知道ConcurrentHashMap 的内部实现原理，而且几乎是个硬性要求了。后一个问题似乎更难一些，主要是进一步考察面试者对细节的一些思考。

- 可以追问一些技术实现细节，比如为什么HashMap中bucket的大小为什么是2的幂之类的实现细节.

>这里的h是”int hash = hash(key.hashCode());”, 也就是根据key的hashCode再次进行一次hash操作计算出来的 . 
length是Entry数组的长度 . 
一般我们利用hash码, 计算出在一个数组的索引, 常用方式是”h % length”, 也就是求余的方式 . 
可能是这种方式效率不高, SUN大师们发现, “当容量一定是2^n时，h & (length - 1) == h % length” . 按位运算特别快 . 


- HashSet的底层原理

new一个hashset时,就是new了一个hashmap

```java
HashSet hashSet = new HashSet();   
hashSet.add("1");
public HashSet() { 
     map = new HashMap<>(); 
     }
```
当我们调用HashSet的add方法时,实际是向hashmap增加了一行键值对,key就是HashSet的对象,value就是Object类型的常量.

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

- ArrayList与Vector的区别
既然说到了同步，那我们面试时的一个经典问题就是ArrayList和Vector的区别？

不要说vector是线程安全的，要说vector在add的时候使用了同步函数，方法上加上了synchronized关键字，ArrayList的add方法是没有加上这个关键字的。当存储空间不足的时候，ArrayList默认增加为原来的50%，Vector默认增加为原来的一倍。


- 阿里面试题
![](https://img-blog.csdn.net/20180406223843235)