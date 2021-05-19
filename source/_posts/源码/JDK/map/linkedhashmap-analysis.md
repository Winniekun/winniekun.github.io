---
title: LinkedHashMap源码解读
date: 2020-06-20 18:38:04
tags:
categories:
thumbnail:
---

## LinkedHashMap源码解读

<!--more-->

## 依赖

![LinkedHashMap依赖](https://i.loli.net/2020/06/20/ibosFG2Q1XYOkmh.png)



LinkedHashMap继承HashMap, 拥有HashMap的所有特性, 并且还额外增加了按一定顺序访问的功能

## 概述

LinkedHashMap 继承自 HashMap，在 HashMap 基础上，通过维护一条双向链表，解决了 HashMap 不能随时保持遍历顺序和插入顺序一致的问题。除此之外，LinkedHashMap 对访问顺序也提供了相关支持。在一些场景下，该特性很有用，比如缓存。在实现上，LinkedHashMap 很多方法直接继承自 HashMap，仅为维护双向链表覆写了部分方法。

## 节点的构造

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

<img src="https://i.loli.net/2021/05/04/PaD7A1jliEQF45q.png" alt="image-20210504171557397" style="zoom:50%;" />

LinkedHashMap内部的Entry相较于Node节点多了两个before、after引用，用来维护LinkedHashMap内部元素的顺序(其内部使用双向链表)。这些还好理解，不过比较好奇的是，为什么HashMap内部的TreeNode节点反而继承的是Entry，而不是HashMap内部的Node

> TreeNode 继承 LinkedHashMap 的内部类 Entry 后，就具备了和其他 Entry 一起组成链表的能力。但是这里会有一个疑问。当我们使用 HashMap 时，TreeNode 并不需要具备组成链表能力。如果继承 LinkedHashMap 内部类 Entry ，TreeNode 就多了两个用不到的引用，这样做不是会浪费空间吗？简单说明一下这个问题（水平有限，不保证完全正确），这里这么做确实会浪费空间，但与 TreeNode 通过继承获取的组成链表的能力相比，这点浪费是值得的。在 HashMap 的设计思路注释中，有这样一段话：
>
> > Because TreeNodes are about twice the size of regular nodes, we
> > use them only when bins contain enough nodes to warrant use
> > (see TREEIFY_THRESHOLD). And when they become too small (due to
> > removal or resizing) they are converted back to plain bins. In
> > usages with well-distributed user hashCodes, tree bins are
> > rarely used.
> >
> > TreeNode 对象的大小约是普通 Node 对象的2倍，我们仅在桶（bin）中包含足够多的节点时再使用。当桶中的节点数量变少时（取决于删除和扩容），TreeNode 会被转成 Node。当用户实现的 hashCode 方法具有良好分布性时，树类型的桶将会很少被使用。
>
> 通过上面的注释，我们可以了解到。一般情况下，只要 hashCode 的实现不糟糕，Node 组成的链表很少会被转成由 TreeNode 组成的红黑树。也就是说 TreeNode 使用的并不多，浪费那点空间是可接受的。假如 TreeNode 机制继承自 Node 类，那么它要想具备组成链表的能力，就需要 Node 去继承 LinkedHashMap 的内部类 Entry。这个时候就得不偿失了，浪费很多空间去获取不一定用得到的能力。

## 类的属性

```java
/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;
/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;
/**
 * The iteration ordering method for this linked hash map: <tt>true</tt>
 * for access-order, <tt>false</tt> for insertion-order.
 *
 * @serial
 */
final boolean accessOrder;
```

## 构造函数

```java
public LinkedHashMap() {
    super();
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}

public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}

public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}

```

和HashMap的构造函数是类似的，不过可以通过入参`accessOrder`来判断内部维护什么顺序的双向链表。

## 增

链表的建立过程是在插入键值对节点时开始的，初始情况下，让 LinkedHashMap 的 head 和 tail 引用同时指向新节点，链表就算建立起来了。随后不断有新节点插入，通过将新节点接在 tail 引用指向节点的后面，即可实现链表的更新。

因为`LinkedHashMap`继承了`HashMap	`，其并未整体重写父类的put操作，而是直接使用父类的put操作，那么问题来了，`LinkedHashMap`

是怎么做到在未重写父类方法情况下，有使得内部的节点具有链表的能力呢？研读源码

### putVal()

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // 对应table数组
    Node<K,V>[] tab;
    // 对应位置的 Node 节点
    Node<K,V> p; 
    // n table的长度
    // 原tab中对应放入Node的位置
    int n, i;
    // 如果 table 未初始化，或者容量为 0 ，则进行扩容
    // (第一次见这种赋值和判断融合的操作，学到了学到了)
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 如果对应的位置的Node节点为空，则直接创建 Node 节点即可。
    // (n-1)&hash 获取所在table的位置
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // 哈希冲突解决
        // key 在 HashMap 对应的老节点
        Node<K,V> e; 
        K k;
        // hash相等 key相等 执行覆盖
        if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 为红黑树的节点
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 为Node节点，则说明是链表，且不是覆盖操作。需要遍历查找
        else {
            // 判断什么时候进行链表转红黑树，什么时候红黑树转链表
            for (int binCount = 0; ; ++binCount) {
                // 如果链表遍历完了都没有找到相同key的元素，说明该key对应的元素不存在，则在链表                   // 最后插入一个新节点
                if ((e = p.next) == null) {
                    // ===1⃣️核心点===
                    // newNode()
                    p.next = newNode(hash, key, value, null);
                    // 链表的长度如果数量达到 TREEIFY_THRESHOLD（8）时，则进行树化。
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 如果待插入的key在链表中找到了，则退出循环
                if (e.hash == hash&&((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 覆盖操作， 也就是相同的key的value进行覆盖
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // onlyIfAbsent进行判断是否需要覆盖oldValue
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 如果超过阀值，则进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}


// LinkedHashMap中，重写了HashMap的`newNode()`方法
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    // 重写了newNode方法
    // 使得返回的节点为Entry节点，具有before、next指针
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}
// 将最后新的节点加入到双向链表的尾部
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null)
        head = p;
    else {
        p.before = last;
        last.after = p;
    }
}
```





### 总结

<img src="/Users/weikunkun/Library/Application Support/typora-user-images/image-20210504175243058.png" alt="image-20210504175243058" style="zoom:33%;" />

整体来看，就是上述的方式，在整个putVal方法中，重写了newNode()方法，同时newNode方法中，LinkedHashMap还执行了自定义的linkedNodeLast()方法。

**小知识点**

通过阅读putVal方法，会发现还有三个after开头的方法

```java
// Callbacks to allow LinkedHashMap post-actions
// 回调方法，供LinekdHash执行一些后置处理
// 在hashmap中并未做出具体的实现
// 但是在linkedhashmap中做出了具体的实现
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```

## 删

和插入操作一样，LinkedHashMap的删除操作也是直接调用父类的方法，不过父类的删除逻辑并不会修复LinkedHashMap内部维护的双向链表，这也不是父类删除方法的职责，所以LinkedHashMap是如何实现在删除过程中维护内部的双向链表的呢？上源码

###  removeNode(int hash, Object key, Object value,boolean matchValue, boolean movable)

```java
public boolean remove(Object key, Object value) {
    return removeNode(hash(key), key, value, true, true) != null;
}

final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    // 这些和增里面是差不多的
    // tab：桶数组 p：待删除节点的前驱节点 n: 桶数组大小 index: 桶数组第i个位置
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null&&(n = tab.length)>0 &&(p=tab[index=(n-1)&hash])!= null) {
        // node为需要删除的节点
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        // 找到对应的key进行remove核心
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            // 如果node ==  p，说明是链表头是待删除节点
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            // 调用删除回调方法进行后续操作
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}

// LinkedHashMap中afterNodeRemoval具体实现
// 双向链表的正常删除操作
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}

// linkedlist的删除操作
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next; // 后继节点
    final Node<E> prev = x.prev; // 前驱节点
		// 删除的为第一个节点
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }
		// 删除的为最后一个节点
    if (next == null) {
        last = prev;
     } else {
        next.prev = prev;
        x.next = null;
     }
}
```

说白了就是双向链表的删除，具体删除逻辑和LinkedList是一样的，先确定当前节点的前驱、后继节点，之后先修改指向后断链防止整个链表断开，在此期间需要判断下要删除的节点是否为第一个节点、最后一个节点，防止爆NPE

### 总结

删除的过程并不复杂，上面这么多代码其实就做了三件事：

1. 根据 hash 定位到桶位置
2. 遍历链表或调用红黑树相关的删除方法
3. 从 LinkedHashMap 维护的双链表中移除要删除的节点

## 查

因为LinkedHashMap内部有两种机制维护内部的双向链表（**按照插入顺序、按照访问顺序**）前面说了插入顺序的实现，接下来研究下访问顺序。默认情况下，LinkedHashMap 是按插入顺序维护链表。不过我们可以在初始化 LinkedHashMap，指定 accessOrder 参数为 true，即可让它按访问顺序维护链表。

### 插入顺序、访问顺序的区别

**插入顺序**:是指LinkedHashMap在数据插入时的插入顺序

1. 比如说1,2,3,4...数据依次从小到大插入
2. 若按照插入顺序输出,输出结果就是1,2,3,4...

**访问顺序**:则是说同样按照插入1,2,3,4...从小到大有序的插入
	1. 如果在插入后你随机访问了某个元素,那么那个元素则会排列到集合的最后一位

```java
@Test
public void insetAndVisitedOrder() {
    Map<String, String> map = orderMap();
    System.out.println(map);
    System.out.println("=================");
    map.get("key-3");
    System.out.println(map);
}
public Map<String, String> orderMap() {
    Map<String, String> map = new LinkedHashMap<>(8, 0.75F, true);
    for (int i = 0; i < 5; i++) {
        map.put("key-" + i, "value-" + i);
    }
    return map;
}

// {key-0=value-0, key-1=value-1, key-2=value-2, key-3=value-3, key-4=value-4}
// =================
// {key-0=value-0, key-1=value-1, key-2=value-2, key-4=value-4, key-3=value-3}
```

阐述完毕，接下来看下其内部如何维护这两种类型的访问顺序

### get(Object key)

```java
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    // 如果 accessOrder 为 true，则调用 afterNodeAccess 将被访问节点移动到链表最后
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}

void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    // 不是尾部节点
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

整体的操作共为两步

1. 删除当前节点所在的位置
2. 将当前节点插入到双向链表的尾部
3. 注意可能当前节点可能是head节点、tail节点

同理，因为是维护了访问的双向链表，所以在对外提供的API，包含有访问性质的，都会调用`afterNodeAccess`来维护该双向链表

如：

1. get
2. getOrDefault

![image-20210504201032288](https://i.loli.net/2021/05/04/18nepvdhPmKiTMg.png)

## 应用

[LRU缓存操作](https://leetcode-cn.com/problems/lru-cache/)

在阐述缓存实现之前，还有一个`afterNodeInsertion`没有讲解

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

// 移除最近最少被访问条件之一，通过覆盖此方法可实现不同策略的缓存
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

> 上面的源码的核心逻辑在一般情况下都不会被执行，所以之前并没有进行分析。上面的代码做的事情比较简单，就是通过一些条件，判断是否移除最近最少被访问的节点。看到这里，大家应该知道上面两个方法的用途了。当我们基于 LinkedHashMap 实现缓存时，通过覆写`removeEldestEntry`方法可以实现自定义策略的 LRU 缓存。比如我们可以根据节点数量判断是否移除最近最少被访问的节点，或者根据节点的存活时间判断是否移除该节点等

```java
public class SimpleCache<K, V> extends LinkedHashMap<K, V> {

    private static final int MAX_NODE_NUM = 100;

    private int limit;

    public SimpleCache() {
        this(MAX_NODE_NUM);
    }

    public SimpleCache(int limit) {
        super(limit, 0.75f, true);
        this.limit = limit;
    }

    public V save(K key, V val) {
        return put(key, val);
    }

    public V getOne(K key) {
        return get(key);
    }

    public boolean exists(K key) {
        return containsKey(key);
    }
    
    /**
     * 判断节点数是否超限
     * @param eldest
     * @return 超限返回 true，否则返回 false
     */
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > limit;
    }
}
```



## 总结

1. LinkedHashMap继承自HashMap，具有HashMap的所有特性
2. LinkedHashMap内部维护了一个双向链表，来维护节点直接的顺序
3. 如果accessOrder为false，则可以按插入元素的顺序遍历元素
4. 如果accessOrder为true， 则可以按访问元素的顺序遍历元素
5. LinkedHashMap的实现非常精妙，很多方法都是在HashMap中留的钩子（Hook），直接实现这些Hook就可以实现对应的功能了，并不需要再重写put()等方法
6. 默认的LinkedHashMap并不会移除旧元素，如果需要移除旧元素，则需要重写removeEldestEntry()方法设定移除策略；
7. LinkedHashMap可以用来实现LRU缓存淘汰策略；

## references

* [LinkedHashMap源码详细分析（JDK1.8）](http://www.tianxiaobo.com/2018/01/24/LinkedHashMap-%E6%BA%90%E7%A0%81%E8%AF%A6%E7%BB%86%E5%88%86%E6%9E%90%EF%BC%88JDK1-8%EF%BC%89/)