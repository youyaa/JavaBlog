> HashMap作为我们日常使用的最频繁的一类容器，我们有必要了解其内部的实现原理。除此之外，HashMap也极受面试官的喜爱，几乎成了面试必问。


## 什么是HashMap
HashMap是一种基于hash算法的key-value键值对形式的非同步容器，允许插入key和value为null的数据。

## HashMap的底层分析

HashMap，是一种根据关键码值（Key）去访问的value一种数据结构，其原理就是：将key通过某种函数映射到数组中的某个位置来存储对应的value，来实现快速精确的查找，这个函数我们就称它为**散列函数**，这样做的好处很明显，避免了线性搜索，但是有一个问题：万一两个不同的Key通过Hash函数计算出来的值是一样的怎么办？我们到底应该怎么去存储这些数据呢？这就引申出了HashMap的特点：

1. Hash函数计算出来的值应该比较平均，比如我们如果用一个70个元素大小的数组来存储60个数据，那么计算出来的结果应该比较能平均地分配到这70个位置上，而不是集中在某几个位置，所以JDK除了对Key的生成hashCode之外，额外还进行了一次Hash操作。
2. 我们可以考虑这样一个问题：我们要存储的数据是无限多的，但是我们的HashMap最终是存储在硬件设备上，硬件上的存储是不可能无限大的，所以无论一个Hash函数多么好，总是避免不了两个不同的Key通过Hash函数计算出来的值是一样的，即发生了**Hash冲突**，那么该怎么解决呢？
   - 拉链法：如果发生了Hash冲突，那么就在冲突的位置上形成一个链表。
   - 开放定址法：如果发生了Hash冲突，那么就往下搜索，找到数组中下一个没有冲突的位置。
3. 假如我们现在有100个数据需要存储，那么我们需要多大的数组来存储呢？1000个的话，相当于大部分的内存被浪费了，10个呢？那么平均每个位置就会存储10个数据，即形成了长度为10的链表，那么搜索的效率又下降了，所以是不是应该有一个适中的大小，这就有了**负载因子**的概念，比如：JDK中的HashMap中的负载因子是0.75，意思是假如一个用一个总长度为100的数组去存储数据，当存储的元素个数大于100*0.75时，表示此时Hash冲突可能比较多，可以考虑扩大数组的大小，以加快搜索的效率，负载因子一般是在时间和空间之间权衡的结果。

### 三个非常关键的点

- equals：在Object类的equals方法是和(==)一样的:

```java
 public boolean equals(Object obj) {
        return (this == obj);
    }
```
但是子类可以对该方法进行重写,以实现自己所需的功能：例如String类就对该方法进行了重写，用于判断两个字符串是否相等。在JDK中的HashMap中，如果发生了Hash冲突，那么在链表上进行遍历的时候，就要判断每个链表节点上的数据中的key是否和要查找的key相同。

- 等号(==)：判断两个对象是否相等，即判断两个对象在内存当中的地址是否相等。

- HashCode：HashCode并不是对象的内存地址，而是利用hash算法，对对象实例的一种具体的描述（或者说对象存储位置的hash算法映射）。



## HashMap常见的面试题

- **HashMap如何减少Hash冲突？为什么要进行两次Hash操作？**

  所谓两次Hash操作是指：通过key获取hashCode以及另外一次rehash操作

  ```js
  位运算：
  &      与        两个位同时为1才为1 
  |      或        两个位同时为0才为0
  ^      异或      两个位相同为0，不同为1
  ~      取反      0变为1，1变为0
  <<     左移      二进制位全部左移，高位丢弃，低位补0
  >>     右移      各二进位全部右移若干位，对无符号数，高位补0。有符号数，各编译器处理方法不一样，有的补符号位（算术右移），有的补0（逻辑右移）
  ```

  ```java
  static final int hash(Object key) {
          int h;
          return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
      }
  ```

  相当于将hashCode的高16位和低16位做一个异或的运算，将落点分布均匀，减少hash冲突。

  

- **平时在使用HashMap时一般使用什么类型的元素作为Key，如何自定义key的类型？**

  使用String或者Integer这样的类。这些类是Immutable(不可变)的，并且这些类已经很规范的覆写了hashCode()以及equals()方法。作为不可变类天生是线程安全的，而且可以很好的优化比如可以缓存hash值，避免重复计算等等。

  >1. 重载hashCode()是为了对同一个key，能得到相同的HashCode，这样HashMap就可以定位到我们指定的桶上。
  >2. 重载equals()是为了向HashMap表明当前对象和key上所保存的对象是相等的，这样我们才真正地获得了这个key所对应的这个键值对。



- **为什么HashMap中bucket的大小为什么是2的幂？**

  这里的h是”int hash = hash(key.hashCode());”, 也就是根据key的hashCode再次进行一次hash操作计算出来的 。length是Entry数组的长度 。
  一般我们利用hash码, 计算出在一个数组的索引, 常用方式是”h % length”, 也就是求余的方式 . 
  可能是这种方式效率不高, SUN大师们发现, “当容量一定是2^n时，h & (length - 1) == h % length” . 按位运算特别快 . 

  

- **HashMap的扩容机制**

  - 扩容：创建一个新的Entry空数组，长度是原数组的2倍。

  - ReHash：遍历原Entry数组，把所有的Entry重新Hash到新数组。

    

- **你知道HashMap中链表的插入方式吗？**

  Java8之前都是头插法，Java8之后就改为了尾插法。

  是因为头插法在多线程环境下扩容时有可能会形成环形链表，get()时会形成死循环，造成CPU100%。



- **为什么HashMap的初始化容量是16？**

  我觉得就是一个经验值，定义16没有很特别的原因，只要是2次幂，其实用 8 和 32 都差不多。

  用16只是因为作者认为16这个初始容量是能符合常用而已。

  

- **Hashmap中的链表大小超过八个时会自动转化为红黑树，当删除小于六时重新变为链表，为啥呢？**

  根据泊松分布，在负载因子默认为0.75的时候，单个hash槽内元素个数为8的概率小于百万分之一，所以将7作为一个分水岭，等于7的时候不转换，大于等于8的时候才进行转换，小于等于6的时候就化为链表。




- **HashSet的底层原理**

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

  

- **ArrayList与Vector的区别**

  既然说到了同步，那我们面试时的一个经典问题就是ArrayList和Vector的区别？

  不要说vector是线程安全的，要说vector在add的时候使用了同步函数，方法上加上了synchronized关键字，ArrayList的add方法是没有加上这个关键字的。当存储空间不足的时候，ArrayList默认增加为原来的50%，Vector默认增加为原来的一倍。



## Hash算法

下面来讲讲Hash算法，Hash算法是一种非常重要的算法，有非常广泛的应用。

### 什么是Hash算法

将任意长度的二进制值串映射为固定长度的二进制值串，这个映射规则就是hash算法，通过原始数据映射生成的二进制串就是hash值。

### Hash算法的要求

1. 从哈希值不能反向推导出原始数据（所以哈希算法也叫单向哈希算法。）
2. 对输入数据非常敏感，哪怕只改1bit，生成的二进制串也有很大的不同。
3. 散列冲突的概率要很小，对于不同的原始数据，哈希值相同的概率非常小。
4. 哈希算法的执行效率要尽量高效，针对较长的文本，也能快速地计算出hash值。

### 常见的Hash算法

1. MD（Message-Digest Algorithm，消息摘要算法)算法：常见的有MD4和MD5算法，MD5（RFC 1321）是 Rivest 于 1991 年对 MD4 的改进版本。它对输入仍以 512 位进行分组，其输出是 128 位。MD5 比 MD4 更加安全，但过程更加复杂，计算速度要慢一点。

2. SHA(Secure Hash Algorithm，安全散列算法)算法：常见的有SHA-224、SHA-256、SHA-384，和 SHA-512 算法（统称为 SHA-2 算法）
3. 国内的 SM3 算法。
4. MurmurHash。

### JDK中String类使用的Hash算法

首先，hashCode()不要求唯一但是要尽可能的均匀分布，而且算法效率要尽可能的快。比如MurMurHash（萌萌哒的名字），JDK中String类的hashCode()的实现：

为什么 h = 31 * h + val[off++]; 这一行使用31 ，而不是别的数字，这是一个魔术吗？

算法即计算31进制数：如abcd的，计算出来就是a·31^3+b·3^2+c·31+d

> 之所以使用 31， 是因为他是一个奇素数。如果乘数是偶数，并且乘法溢出的话，信息就会丢失，因为与2相乘等价于移位运算（低位补0）。使用素数的好处并不很明显，但是习惯上使用素数来计算散列结果。 31 有个很好的性能，即用移位和减法来代替乘法，可以得到更好的性能： 31 * i == (i << 5） - i， 现代的 VM 可以自动完成这种优化。这个公式可以很简单的推导出来。

### Hash攻击



