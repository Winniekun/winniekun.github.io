---
title: ArrayList源码解读
date: 2020-02-10 09:49:19
tags: 
	- JDK源码
	- List
categories: 源码
thumbnail: https://i.loli.net/2020/03/10/R2ExlKrgYVPeM6U.png
---

## ArrayList 源码解读

<!--more-->

### 依赖

![ArrayList依赖](https://i.loli.net/2020/05/24/fKLA9sY3tM4Ur1V.png)

关于为什么**ArrayList**已经继承了**AbstractList**还要实现**List**接口, 可参考 [Why does LinkedHashSet extend HashSet and implement Set](https://stackoverflow.com/questions/2165204/why-does-linkedhashsete-extend-hashsete-and-implement-sete)

![StackOverflow解释](https://i.loli.net/2020/05/24/8qwE9WoJ4MUyvNx.png)

可能这样增加可读性吧, 接口的[第二个功能一样](https://kongwiki.me/2020/05/13/interface-Abstarct-diff/)

### 字段

```java
 /**
     * Default initial capacity.
     * 默认初始化容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances
     * 空实例的共享数组实例
     * 若是无参构造方法 使用 DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
     * 若是含参构造方法 使用 EMPTY_ELEMENTDATA 
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     * 
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
     
    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     * 某些 VM 保留数组头部用于存储一些 header words
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

可以发现, **elementData** 和 **size** 两个变量在ArrayList的运行中会有很大的变动, 可以使用**反射**进行观察



### 构造函数

#### ArrayList() 

```java
/**
* 构造函数1: 默认构造函数, 
* DEFAULTCAPACITY_EMPTY_ELEMENTDATA = 0,
* 初始是一个空数组, 当添加一个元素的时候, 扩充为10(如何扩充为10, 详见下述)
* 原因详见 transient Object[] elementData上的注释
*/
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

**也就是在创建对象的时候，并没有立刻初始化容器的大小，直到第一次调用 `add()`方法的时候，才进行确定**。



#### ArrayList(int initialCapacity)

```java
/**
* 构造函数2 带初始容量参数的构造函数, (用户自己定义容量)
*/
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

**初始化容量大小为`initialCapactiy`**



#### ArrayList(Collection<? extends E> c)



```java
/**
* Constructs a list containing the elements of the specified
* collection, in the order they are returned by the collection's
* iterator.
*
* @param c the collection whose elements are to be placed into this list
* @throws NullPointerException if the specified collection is null
* 构造包含指定collection元素的列表 这些元素利用该集合的迭代按顺序返回
* 如果指定的集合为null, 抛出 NullPointerException
*/
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

**总结**

1. **常量EMPTY_ELEMENTDATA和DEFAULTCAPACITY_EMPTY_ELEMENTDATA是为了初始化elementData的。如果为无参构造函数，使用DEFAULTCAPACITY_EMPTY_ELEMENTDATA；如果为含参构造函数，使用EMPTY_ELEMENTDATA**
2. 以无参构造方法创建arraylist时, 实际上初始化的值是一个空数组. 添加第一个元素时 扩容为10
3. 如果使用`Collection`实例初始化, **不为空**将调用`toArray()`方法初始化`elementData`, **为空**时初始化为空数组`EMPTY_ELEMENTDATA` 

### 增

#### add(E e)

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

在调用 `add()`方法的时候，首先调用 `ensureCapacityInternal()`方法来**确定是否还有容量来容纳新的值**。`size`是表示数组的初始大小，默认为 0 。

![ArrayList-add执行步骤](https://i.loli.net/2020/05/24/risyWCVd6ED1JYF.png)

**按照流程走一遍:**

1. 第一次调用的时候:
   1. 第一步, 因为还为添加成功元素, `size = 0` , `size + 1 = 1`
   2. 第二步, 首先执行第三步, 传入两个参数`elementData`和`minCapacity`
   3. 第三步,  此时的`elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATAT`, 然后 `minCapaciy==1`, 返回`DEFAULT_CAPACITY`
   4. 第四步,  此时的`minCapacity==10` , `elementData.length = 0` 所以执行`if`分支
   5. 第五步, `oldCapacity==0`  经计算`newCapacity==0`  然后`newCapacity-minCapacity<0`, 所以`newCapacity==10` 之后进行数据的复制，这时，`elementData`的长度是 10.，第一个元素成功的添加到数组中。
2. 第二次调用
   1. 第一步, 因为已经添加成功了一个元素, `size = 1` , `size + 1 = 2`
   2. 第二步, 首先执行第三步, 传入两个参数`elementData`和`minCapacity`
   3. 第三步,  此时的`elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATAT`, 然后 `minCapaciy==2`, 不执行`if`分支, 返回 `2`
   4. 第四步,  此时的`minCapacity==2` , `elementData.length = 10` 所以不执行执行`if`分支, 也就是无需扩容
3. 反复添加直到添加第11个元素的时候, 再次出发**第四步**的**grow**, 进行扩容, 扩为原先的1.5倍.



#### add(int index, E element)

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```

使用**arraycopy**方法, 可以看做将指定坐标位置以及右侧所有元素向后移动一位，腾出空间存放新元素。

#### addAll(Collection<? extends E> c)

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```

#### addAll(int index, Collection<? extends E> c)

```java
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);
    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

#### 总结

1. 共实现四中添加方法:
   1. 添加单个元素(默认在尾部添加)
   2. 在指定位置添加单个元素
   3. 添加一个集合(默认在末尾)
   4. 在指定位置添加一个集合
2. 每次执行add操作时都会首先调用 `ensureCapacityInternal()`方法来**确定是否还有容量来容纳新的值**, 若是没有则进行扩容, 有则直接添加



### 删

`remove()`方法用来从集合中删除一个元素，可以删除指定索引出的元素，也可以指定元素。

#### remove(Object o)

```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
// 将index位置后的所有元素前移一位, 之后size-1 
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

**流程走一遍**

```java
ArrayList<String> arrayList = new ArrayList<>();
arrayList.add("a");
arrayList.add("b");
arrayList.add("c");
arrayList.remove("a");
```

1. 要删除`a`这个元素，把 `a`传入到 `remove()`这个方法，首先进行判断是否为`null`，因为 `ArrayList`可以存放 `null`，如果为 `null` 则用`==`判断元素是否相等，否则用 `equals()`方法进行判断。

2. 然后循环遍历 `elementData`这个数组，`size`为数组元素个数，找到和 `a`相等的元素的索引，例子中 index = 0， 然后把 0 传入到 `fastRemove()`方法中，之后计算要移动的元素的个数：

   ```tex
   int numMoved = size - index -1;
   ```

   例子中  `numMoved = 3 - 0 -1 = 2`，也就是一共要移动两个元素，然后把 `a`之后的两个元素往前移动，然后进行 `size-1`，把最后一个元素位置的值设置为 null。

   这样即可在`ArrayList`中删除一个元素。



#### remove(int index)

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```

#### 总结

没啥好总结的

### 查

```java
public E get(int index) {
    // 判断查询的位置是否符合要求
    rangeCheck(index);
    return elementData(index);
}

private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

E elementData(int index) {
    return (E) elementData[index];
}
```

没啥好说的, 就是正常数据的使用

### 附加

#### hugeCapacity 方法

```java
   private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

只有`newCapacity` >`MAX_ARRAY_SIZE` 才回调用该方法, 方法中使用三元运算, 意思是若果`minCapacity`> `MAX_ARRAY_SIZE`   则`newCapacity`的容量大小为`Integer.MAX_VALUE`也就是2**31次方, 若是`minCapacity`< `MAX_ARRAY_SIZE`   则`newCapacity`的容量大小为`Integer.MAX_VALUE-8`



#### ensureCapacity用法

Arraylist源码中有一个ensureCapacity方法

```java
/***
* 如有必要，增加此 ArrayList 实例的容量，
* 以确保它至少可以容纳由minimum capacity参数指定的元素数。
*/
 public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
```

**该方法适用于添加大量的元素, 以减少上文的增量重新分配的次数**

```java
 ArrayList<Object> list = new ArrayList<>();
        final int N = 10000000;
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < N; i++) {
            list.add(i);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("使用ensureCapaticy方法之前: "+ (endTime - startTime));

        list = new ArrayList<>();
        list.ensureCapacity(N);
        startTime = System.currentTimeMillis();
        for (int i = 0; i < N; i++) {
            list.add(i);

        }

        endTime = System.currentTimeMillis();
        System.out.println("使用ensureCapaticy方法之后: "+ (endTime -startTime));


```

```
使用ensureCapaticy方法之前: 3345
使用ensureCapaticy方法之后: 257
```

## 总结

`ArrayList`集合底层是用一个 Object 数组来存放元素的，其可以存放 null 值，因为是使用数组来存放元素，所以在知道索引的情况下，进行元素的查找是很快的，但是也有缺点，如果数组的容量不能够存放新元素的时候，会进行数组的扩容，也就是把数组元素复制到一个容量更大的数组中，所以如果在经常进行元素添加和删除操作的情况下效率会比较低。还有一点，`ArrayList`不是线程安全的，要保证线程安全，可以使用 `Vector`代替。

### System.arraycopy(src, srcPos, dest, destPos, length) 与 Arrays.copyOf(original, newLength)区别

arraycopy()的参数

1. src:源数组;
2. srcPos:源数组要复制的起始位置;
3. dest:目的数组;
4. destPos:目的数组放置的起始位置;
5. length:复制的长度.

两者的区别在于，Arrays.copyOf()不仅仅只是拷贝数组中的元素，在拷贝元素时，会创建一个新的数组对象。而System.arrayCopy只拷贝已经存在数组元素, **是否创建新的数据对象,根据写法而定**

```
//System.arraycopy,只拷贝已存在的数组元素
int[] src = {0, 1, 2};
int[] dest = new int[3];
System.arraycopy(src, 0, dest, 0, src.length);
System.out.println(Arrays.toString(dest));      //[0, 1, 2]
//Arrays.copyOf,会创建一个新的数组对象
int[] src = {0, 1, 2};
int[] dest = Arrays.copyOf(src, src.length);
System.out.println(Arrays.toString(dest));    //[0, 1, 2]
```

实际上Arrays.copyOf方法的内部实现也是通过System.arraycopy方法实现，在Arrays类中有多个copyOf的重载方法，现以拷贝int[]为例：

```
1 public static int[] copyOf(int[] original, int newLength) {
2     int[] copy = new int[newLength];
3     System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
4     return copy;
5 } 
```





## References

* [ArrayList源码](https://diguage.github.io/jdk-source-analysis/#_arraylist)
* [ArrayList源码分析](https://my.oschina.net/mengyuankan/blog/885590/print)