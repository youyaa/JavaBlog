## Heap(堆)参数设置

-Xmx  设置堆的最大值，等价于MaxHeapSize。

-Xms  设置堆的最小值，也叫初始值，等价于InitialHeapSize。

> 1. MaxHeapSize必须大于等于InitialHeapSize，否则启动会报错。
> 2. InitialHeapSize必须大于等于1M，否则启动也会报错。
> 3. HeapSize是2M对齐的，如果设置的大小不是2M的倍数，将会自动转换成2M的倍数整数。

## Heap Size的动态调整

-MinHeapFreeRatio  

-MaxHeapFreeRatio

都是百分比。GC之后是否对堆的内存进行扩容和缩容。

默认情况下，MinHeapFreeRatio=40，MaxHeapFreeRatio=70

> CMS GC下，如果没有指定老年代固定使用率触发CMS GC的阈值，那么MinHeapFreeRatio会配合CMSTriggerRatio(默认80)参数计算出触发CMS GC的老年代使用阈值，具体算法是:
>
> ((100-MinHeapFreeRatio)+CMSTriggerRatio*MinHeapFreeRatio/100)/100，即CMS默认在老年代使用达到92%时，触发GC。

## 堆的New Size(新生代)参数设置

-Xmn   等价于同时设置了NewSize和MaxNewSize，并且值都相等，例如-Xmn128M，

等同于-XX:NewSize=128M -XX:MaxNewSize=128M

-NewSize    设置新生代有效内存的初始化大小，也可以说是新生代有效内存的最小值，当新生代回收之后有效内存可能会进行缩容，这个参数就指定了能缩小到的最小值。

-MaxNewSize    就是设置新生代有效内存的最大值，当对新生代进行回收之后可能会对新生代的有效内存进行扩容，那到底能扩容到多大，这就是最大值。

-NewRatio    NewRatio本意表示当前老生代可用内存/当前新生代可用内存的比值，默认是2，不过真正运行的时候新老生代的有效内存不一定是这个比值，某些条件下成立的。

## 堆的Perm Size(永久代)参数设置  -JDK1.8之前

-XX:PermSize  永久代大小。
-XX:MaxPermSize  永久代最大size。

