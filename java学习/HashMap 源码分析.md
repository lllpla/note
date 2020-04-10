#HashMap 源码分析



## 一、概览

本文使用 JDK1.8,

我们先借助IDE对HashMap储存key-value的结构有个大致的了解

先生成一个有100个数据的map，key是我自定义的对象，value是一个随机 Integer，然后用 IDE 透视：

![img](https://cdn.yuque.com/yuque/0/2018/png/102759/1530106911519-98fc434c-62c3-4262-ac65-b5f19c8c3c61.png)

可以看到，最主要的是table，和一些设置的数值。

map最核心的结构是 table，是一个node的数组，node里面存着 key和value，而node本身是一个链表。或者是一个红黑树的节点 TreeNode。

![img](https://cdn.yuque.com/yuque/0/2018/png/102759/1530106480489-9e5ec110-8748-4577-a934-4c4ec567851b.png)

我们用图直观的看一下：

![img](https://cdn.yuque.com/yuque/0/2018/png/102759/1529915531228-11fb7529-d2b1-48fc-bfc8-a65697fda608.png)

这个样例的map的生成方法在扩展5，读完本文，你或许也可以做一个高度碰撞的HashMap。

## 二、构造函数

我们常用的 HashMap 构造方法

```java
// 无参构造方法
HashMap<String, String> map1 = new HashMap<String, String>();
// 含参数构造方法
HashMap<String, String> map2 = new HashMap<String, String>(16);
HashMap<String, String> map3 = new HashMap<String, String>(16,0.75F);
```

查看源代码

```java
// HashMap 类定义
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable

// HashMap 构造函数
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    // note: 设置 loadFactor 为 0.75 ，没啥好说的
}

// HashMap 含参构造函数 (int)
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
    // note: 执行下面一个函数，附带的 第二个参数 为 0.75
}

// HashMap 含参构造函数 (int, float)
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    // note: 良好的检查，负数第一时间抛出错误
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // note: 防止超过最大容量，最大容量值为 2^30
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    // note: 检查 loadFactor 大于0，且不为 NaN
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
    // note:调用了 tableSizeFor(initialCapacity) ，
    // 根据 initialCapacity 计算 threshold
}
```

用到了几个变量，下面提到可以再回来找

| K                   | key的泛型类型                                                |
| ------------------- | ------------------------------------------------------------ |
| V                   | value的泛型类型                                              |
| float loadFactor    | 负载因子，默认是0.75                                         |
| int initialCapacity | 初始容量，用来计算 threshold                                 |
| int threshold       | 扩容门槛，默认是0如果 HashMap 内数据超过 threshold * loadFactor则会进行扩容操作。 |

无参数的构造方法比较平常，构造函数为其设置了 loadFactor ，而另外两个构造函数则设置了 initialCapacity 和 自定的 loadFactor。当传入  initialCapacity 之后，通过一个函数，我们最终计算出最重要的 threshold 属性。来看下这个函数，主要是通过位运算，让 threshold 始终是 2 的 平方数（方便扩容时重新落位）

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1; 
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

这里有个技巧，如果 HashMap 大小已知，则建议预设好threshold和loadFactor。防止map多次扩容（默认为0）。

如现有100个数据，为防止扩容，则先计算扩容临界值 100/loadFactor(默认0.75) = 133，则 initialCapacity 应设置为 2 ^ 8 = 256

## 三、put()方法

平时用法

```java
map.put("test","test");
```

debug之，note里面如果不知道变量是什么，可以查看代码下方的变量表

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// note：这里先执行了 hash() 方法
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    // 一般情况，使用 对象的 HashCode 计算
    // 算法是 hash = hashcode ^ (hashcode >>> 16)
}

// 返回第一个方法，执行 putVal()方法
// 返回：如果被替换，返回旧值。如果未被替换，返回null
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        // 特别注意 hash = hash(key.hashCode)，并非原来的 hashCode
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 检查table有无初始化，第一次运行会到这里，进行 resize() 操作
        // 这个操作后面会讲，这里这用来初始化 table 数组
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        // 用i = (n - 1) & hash 计算 node 应该放在哪个槽位
        // 这是一种对 hash 取余数的方法，取的结果会分布在 table 的长度之内
        // 如果这个地方是 null
        // 那么，把key,value包装成 node，存入table[i]
        else {
        // 如果这个地方原来已经有一个 node 
            Node<K,V> e; K k;
            if (p.hash == hash && // note: 先检查旧的新的node的hash是否相等
                ((k = p.key) == key || (key != null && key.equals(k))))
            // note: 检查旧key和和key的hashcode是否相等 ，或者旧key和新key是 equal 的
                e = p;
            // note: 直接用新的node替换掉旧的node，跳转到 if (e != null)
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // note: 如果旧的note已经升级为 TreeNode ，那么用红黑树的方法来做
            // 默认情况下，有多个node都在一个槽位的情况，是用链表的数据结构
            // 但如果同一个槽位有超过8个node，则会升级为红黑树
            else {
            // 这是默认情况的分支，有多个 node 都在同一个槽位，但是 hashcode 不相同
                for (int binCount = 0; ; ++binCount) {
                    // 写一个死循环
                    if ((e = p.next) == null) {
                        // 一直到达 p.next 为 null
                        p.next = newNode(hash, key, value, null);
                        // 插入一个新 node
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        // 如果超过红黑树转换阈值，则转换为红黑树
                        break;
                    }
                    // 如果多个 node 的链表中，有一个 node 是和新node的key是一样的
                    // 结束循环，跳转到下面 if (e != null)
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    // p = p.next 遍历链表
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                // 从上面跳转而来，判断是否要用新值替换旧值
                V oldValue = e.value;
                
                if (!onlyIfAbsent || oldValue == null)
                // 用新值替换掉旧值，if判断可以设置不替换
                    e.value = value;
                // 钩子，没啥用
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
        // 判断是否需要扩容
            resize();
        // 钩子，没啥用
        afterNodeInsertion(evict);
        return null;
    }
```

这里用到了一些变量

| key                                 | put方法的传入 key 值                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| value                               | put方法传入的 value 值                                       |
| hashcode                            | 对象的hashcode                                               |
| hash                                | 对hashcode进行了二次计算，缩小了hashcode的大小hash方法的意义在于，使得hashcode更加均匀，使hash值的位值在高低位上尽量分布均匀 |
| Node<K,V> implements Map.Entry<K,V> | `static class Node<K,V> implements Map.Entry<K,V> {     final int hash;     final K key;     V value;     Node<K,V> next;      Node(int hash, K key, V value, Node<K,V> next) {         this.hash = hash;         this.key = key;         this.value = value;         this.next = next;     } `key/value结构将被包装成Node这个数据结构 |
| Node<K,V>[] table                   | 存放数据的Node的数组                                         |

hash和槽位的算法：

高16bit不变，低16bit和高16bit做了一个异或，可以在资源消耗比较低的情况下，打乱一下 hash，让计算出来的槽位下标更加均匀。通过hashCode()的高16位异或低16位实现的：`(h = k.hashCode()) ^ (h >>> 16)`，主要是从速度、功效、质量来考虑的，这么做可以在bucket的n比较小的时候，也能保证考虑到高低bit都参与到hash的计算中，同时不会有太大的开销

![img](https://cdn.yuque.com/yuque/0/2018/png/102759/1529994464889-970160fc-068f-404d-b64c-2eac9daf7889.png)

## 四、get()方法

平时用法

```java
map.get("test");
```

源码

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
    // 计算 hash，调用 getNode 方法取出 node，返回 node.value
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 获取到hash对应槽位的node
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
        // 检查 hashcode 是否相同，相同返回 这个 node，不相同继续遍历链表
            return first;
        if ((e = first.next) != null) {
            // 判断红黑树的情况
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 遍历链表，取出hashcode相同的node
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
        return null;
    }
```

## 五、resize()方法

这是一个 HashMap 的私有方法，是非常重要的方法

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        // 暂存原来的 table 
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        // 暂存原来的长度，为null说明是刚刚初始化，table还没有新建
        int oldThr = threshold;
        // 暂存原来的threshold
        int newCap, newThr = 0;
        if (oldCap > 0) {
            // 原来的 table 已经初始化走这里，是正常扩容
            if (oldCap >= MAXIMUM_CAPACITY) {
                // 超过最大值，不再扩容，碰撞的几率会增加
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)    
                // 扩容两倍（原理见后）
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            // 尚未初始化，但已经指定了初始大小
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            // 默认初始化，用默认值计算 threshold 和 newThr
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 如果没计算 newThr，现在再补算一次
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        // 更新table，是一个空的table，容量是原来的两倍
        if (oldTab != null) {
            // 如果原来有数据，走这里，遍历旧的table，拉出来重新插入
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    // 原槽位存到e，并且不为空走这里
                    oldTab[j] = null;
                    // 清空原槽位，等回收
                    if (e.next == null)
                    // 如果这里不是链表（只有一个节点），直接重新落位
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                    // 如果是红黑树，单独的分裂逻辑
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        // 原来是链表
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // 遍历链表
                            if ((e.hash & oldCap) == 0) {
                                // 槽位未变，挂到loTail临时链表
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                // 槽位变成原槽位+oldCap
                                // 挂到 hiTail 临时链表
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 把原链表分裂成两个链表，然后分别扔到对应的槽位
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        // 如果原来没有数据，直接返回
        return newTab;
    }
```

两倍扩容的算法

因为是两倍扩容，n-1 永远 是 1 组成，所以重新计算槽位时，要么是在前面 加个 0，要么是前面加个1

![img](https://cdn.yuque.com/yuque/0/2018/png/102759/1529995263382-aa76da9d-0b64-499a-979f-6b2cb3c86921.png)

![img](https://cdn.yuque.com/yuque/0/2018/png/102759/1529995099397-5b986449-3f3f-4867-ab55-75147e1cf3c0.png)

如果是加0，那么槽位不动，如果是加1，那么等于移动了一个原容量

![img](https://cdn.yuque.com/yuque/0/2018/png/102759/1529995376543-ee09bc1e-0f08-4d43-a9a8-b3ddb64d8ddb.png)

移动的过程相当于全部重新插入，所以是比较费时的

![img](https://cdn.yuque.com/yuque/0/2018/png/102759/1529995435694-23c26101-4f04-408b-8f72-676828f93abd.png)

## 六、扩展

### 6.1 Hashcode的计算规则

HashMap中，hash由hashcode计算得出，那么对象的hashcode如何计算呢？

1. Object类的hashCode.返回对象的内存地址经过处理后的结构，由于每个对象的内存地址都不一样，所以哈希码也不一样。这个是native方法，这个取决于JVM的设计，一般是某种地址的偏移。
2. String类的hashCode.根据String类包含的字符串的内容，根据一种特殊算法返回哈希码，只要字符串的内容相同，返回的哈希码也相同。
3. Integer等包装类，返回的哈希码就是Integer对象里所包含的那个整数的数值，例如Integer i1=new Integer(100),i1.hashCode的值就是100 。由此可见，2个一样大小的Integer对象，返回的哈希码也一样。
4. int，char这样的基础类，它们没有hashCode，如果需要存储时，将进行自动装箱操作，计算方法同上。

### 6.2 HashMap与HashTable的主要区别

他们的主要区别其实就是Table加了线程同步保护

1. HashTable线程更加安全，代价就是因为它粗暴的添加了同步锁，所以会有性能损失。
2. 有更好的concurrentHashMap可以替代HashTable

### 6.3 自定义对象作为 key 有什么要求

必须要重写equals方法和hashCode方法，且放入map之后不要再修改对象

重写hashCode方法，可以使用 Objects.hash(item1, item2, item3) 来多并一

如果不重写，则相同内容的两个对象，如果内存地址不同，则会在map中有两个node

最要命的是，你重新new一个自定义对象查询时(get())，如果内存地址和 map 内的 node 的内存地址不一致，将找不到你之前存的，内容一样的 node

### 6.4 HashSet 和 HashMap有什么异同

HashSet实际上用的是HashMap，HashSet的值是HashMap的key，HashSet在HashMap中的value是 new Object()

```java
// HashSet add方法
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
// PRESENT定义为
private static final Object PRESENT = new Object();
// HashSet contains方法
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

### 6.5 生成一个高度碰撞的HashMap

```java
public class Main {

    public static void main(String[] args) {
        Random random = new Random();
        Map<EasyCollision, Integer> map = Stream.generate(random::nextInt).
                limit(100).collect(Collectors.toMap(EasyCollision::new, Function.identity()));
    }

    static class EasyCollision{
        private Integer hashSource;

        public EasyCollision(Integer hashSource) {
            this.hashSource = hashSource;
        }

        @Override
        public boolean equals(Object o) {
            if (o == null || getClass() != o.getClass()) return false;
            EasyCollision that = (EasyCollision) o;
            return Objects.equals(hashSource, that.hashSource);
        }

        @Override
        public int hashCode() {

            return hashSource % 15;
        }

        @Override
        public String toString() {
            return "EasyCollision{" +
                    "hashSource=" + hashSource +
                    '}';
        }
    }
}
```

 