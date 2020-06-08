> 线程本地存储，即为每个线程单独分配一个内存，不同享。



### Thread类结构分析

在ThreadLocal类中有一个内部类ThreadLocalMap，

但是Thread类持有这个内部类的引用。

```java
public class Thread implements Runnable {
 
//内部持有ThreadLocal类的引用
ThreadLocal.ThreadLocalMap threadLocals = null;
  
  static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
        private static final int INITIAL_CAPACITY = 16;
        private Entry[] table;
  }
}
```

在ThreadLocalMap类中实现了类似HashMap的功能，用一个Entry[]数组来表示每个节点。



### ThreadLocal类结构分析

来看一下ThreadLocal中的方法：

```java
get()：返回此线程局部变量的当前线程副本中的值。
initialValue()：返回此线程局部变量的当前线程的“初始值”。
remove()：移除此线程局部变量当前线程的值。
set(T value)：将此线程局部变量的当前线程副本中的值设置为指定值。
```

### set()方法

```java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

//获取当前Thread中的ThreadLocalMap对象
ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

//当ThreadLocalMap不存在时创建
void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

1. 获取当前线程。
2. 从当前线程中获取ThreadLocalMap。
3. 若ThreadLocalMap存在，则以当前ThreadLocal为key，传入的值为value，存入Map中。
4. 若ThreadLocalMap不存在，则创建后，放入。

> 注意点：当调用map的set()方法时，解决hash冲突的办法是：线性探测，即若对应下标不为空，则继续往下寻找，找到不为空的为止。

### get()方法

```java
 public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }


private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            while (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == key)
                    return e;
                if (k == null)
                    expungeStaleEntry(i);
                else
                    i = nextIndex(i, len);
                e = tab[i];
            }
            return null;
        }
```

1. 获取当前线程。
2. 从当前线程中获取ThreadLocalMap。
3. 从map中获取Entry对象。因为采取了线性探测法，在取Entry的时候，如果根据计算出来的下标未取到，则继续向下寻找。

### remove()方法

```java
public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```

从map中删除。