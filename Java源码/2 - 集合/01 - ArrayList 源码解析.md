## 基础介绍

ArrayList 底层数据结构就是一个数组：

1. index 表示数组下标，从 0 开始计数，elementDatda 表示数组本身
2. `DEFAULT_CAPACITY` 表示**数组的初始化大小，默认是10**
3. size 表示数组的大小，int 类型，**没有使用 volatile 修饰，非线程安全**
4. modCount 统计当前数组被修改的版本次数，数组结构有变动，就会`+1`

### 类注释

1. 允许 put null 值，会自动扩容
2. size、isEmpty、get、set、add 等方法的时间复杂度都是`O(1)`
3. 不是线程安全的，多线程情况下，推荐使用线程安全类：`Collections#synchronizedList`
4. 增强 for 循环，或者使用迭代器迭代过程中，如果数组大小被改变，会快速失败，抛出异常

## 源码解析

### 1、初始化

三种方法：**无参数直接初始化**、**指定大小初始化**、**指定初始数据初始化**。

**源码**

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
// 保存数组的容器，默认是null
transient Object[] elementData;

// 无参数直接初始化
public ArrayList() {
 	 this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 指定大小初始化
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
      this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
      this.elementData = EMPTY_ELEMENTDATA;
    } else {
      throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}

// 指定初始化数据初始化
public ArrayList(Collection<? extends E> c) {
    Object[] a = c.toArray();
  	// 如果给定的集合不为空
    if ((size = a.length) != 0) {
      // 传入的集合类型为ArrayList时，数组直接地址指向a，否则拷贝数据
      if (c.getClass() == ArrayList.class) {
        elementData = a;
      } else {
        elementData = Arrays.copyOf(a, size, Object[].class);
      }
    } else {
      // 给定的集合数据为空时，默认空数组
      elementData = EMPTY_ELEMENTDATA;
    }
}
```

从源码中可以得到以下几点：

1. ArrayList 无参构造器初始化时，**默认大小是空数组**，数组大小 10 是在第一次 add 的时候扩容的数组值。
2. 指定数据初始化的时候，在传入的数据为空时，依旧会初始化为空数组。当传入的集合类型为ArrayList 时，elementData 会直接使用 toArray 生成的数组。

### 2、新增和扩容

添加元素时进行了两步操作：

1. 判断是否需要扩容
2. elementData 数组赋值，这里是**非线程安全的**

```java
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
  // 判断数组的大小是否还够存放新数据，不够则扩容，size是当前数组存放元素的个数
  ensureCapacityInternal(size + 1);  
  // 数组赋值，线程不安全
  elementData[size++] = e;
  return true;
}
```

扩容（ensureCapacityInternal）的源码

```java
private void ensureCapacityInternal(int minCapacity) {
  	ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

// 计算数组需要的最小容量
private static int calculateCapacity(Object[] elementData, int minCapacity) {
  	// 如果是初始化时未指定大小，则使用默认的大小 10
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
      return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
  	// 否则返回存放当前元素需要的最小容量值
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
  	// 数组改变的标志+1
    modCount++;

    // 数组需要的最小容量大于数组的长度，数组需要扩容
    if (minCapacity - elementData.length > 0)
      grow(minCapacity);
}

 /**
  * Increases the capacity to ensure that it can hold at least the
  * number of elements specified by the minimum capacity argument.
  *
  * @param minCapacity the desired minimum capacity
  */
private void grow(int minCapacity) {
    // 记录旧的数组长度
    int oldCapacity = elementData.length;
  	// 新的数组长度等于旧的数组长度的 1.5 倍，oldCapacity >> 1相当于除以 2
    int newCapacity = oldCapacity + (oldCapacity >> 1);
  	// 新的数组长度小于需要的最小容量时，扩容值修改为最小容量值
    if (newCapacity - minCapacity < 0)
      newCapacity = minCapacity;
  	// 新的数组长度超过最大值时，MAX_ARRAY_SIZE 的值为 Integer 表示的最大值-8
    if (newCapacity - MAX_ARRAY_SIZE > 0)
      newCapacity = hugeCapacity(minCapacity);
    // 数组的复制
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
  	// 内存溢出
    if (minCapacity < 0) // overflow
      throw new OutOfMemoryError();
  	// 需要的最小容量超过最大值时，返回Integer 表示的最大值
    return (minCapacity > MAX_ARRAY_SIZE) ?
      Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}
```

Java 的源码有一些需要注意的点：

1. 初始化未指定大小的集合，第一次扩容的大小为 10
2. 添加元素时，不管数组有没有扩容，modCount 都会`+1`
3. 数组容量的最大值是 Integer.MAX_VALUE，超过这个值，JVM 就不会给数组分配内存空间了
4. 新增元素时，没有对值进行严格的校验，所以 ArrayList 可以添加`null`值。

源码中值得我们去学习的地方：

1. 代码书写很优雅，一个方法只做一件事，便于理解
2. 要有边界意识，数组的下标最大不超过 Integer 最大值，最小不能小于 0 

### 3、扩容的本质

ArrayList 中的数组扩容是通过调用 Arrays 的 copyOf 方法。先创建一个符合我们预期容量的新数组，然后把旧数组的元素拷贝过去。通过调用 `System.arraycopy` 方法进行拷贝，此方法是 native 修饰的方法

```java
/**
 * @param src     被拷贝的数组
 * @param srcPos  从数组那里开始
 * @param dest    目标数组
 * @param destPos 从目标数组那个索引位置开始拷贝
 * @param length  拷贝的长度 
 * 此方法是没有返回值的，通过 dest 的引用进行传值
 */
public static native void arraycopy(Object src, int srcPos,
                                    Object dest, int destPos,
                                    int length);
```

### 4、删除

在源码中关于删除的方法：

+ `public boolean remove(Object o) `：删除第一个匹配的元素
+ `public E remove(int index)`：删除指定位置上的元素，并返回该元素
+ `private void fastRemove(int index)`：私有方法，被`remove(Object o)`方法调用
+ `public boolean removeAll(Collection<?> c)`：从此列表中删除包含在指定集合中的所有元素。
+ `public void clear()`：清空集合

```java
/**
 * 移除此列表中指定位置的元素
 */
public E remove(int index) {
    // 数组越界的检查，index 大于 size 时会抛 IndexOutOfBoundsException 异常
    rangeCheck(index);

    modCount++;
    // 当 index 小于 0 时，在这里取值的时会抛出 ArrayIndexOutOfBoundsException 异常 
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
      System.arraycopy(elementData, index+1, elementData, index,
                       numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}

/**
 * 如果指定元素存在，则从此集合中删除第一次出现的指定元素，如果不存在，在保持不变，返回false
 */
public boolean remove(Object o) {
  	// 如果要删除的元素为 null 时
    if (o == null) {
      for (int index = 0; index < size; index++)
        // 判断元素是否为null
        if (elementData[index] == null) {
          fastRemove(index);
          return true;
        }
    } else {
      for (int index = 0; index < size; index++)
        // 使用equals判断元素是否相等
        if (o.equals(elementData[index])) {
          fastRemove(index);
          return true;
        }
    }
    return false;
}

private void fastRemove(int index) {
  	// 记录数组的结构变化
    modCount++;
  	// 需要移动的元素个数
  	// size 从 1 开始计算，index 从 0 开始，所以需要 -1
    int numMoved = size - index - 1;
    if (numMoved > 0)
      System.arraycopy(elementData, index+1, elementData, index,
                       numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

从源码可以看出一些注意点：

1. 新增的时候没有对 null 做校验，删除的时候也是可以删除 null 值的
2. 删除所有元素使用的是`clear()`方法
3. 删除的时候，只做了 index 是否大于 size 的判断，index 为负值时在数组取值时抛出异常。小于 0 和大于 size 时的抛出异常不同。
4. 元素相等的判断在不为 null 的情况下使用 equals 方法，自定义类型元素的删除要注意 equals 的具体实现。

### 5、单向迭代器

实现`java.util.Iterator`接口类，迭代器中定义的几个参数：

```java
int cursor;// 迭代过程中，下一个元素的位置，默认从 0 开始。
int lastRet = -1; // 最后一次迭代时返回的值的索引位置；默认为 -1。
int expectedModCount = modCount;// expectedModCount 表示迭代过程中，期望的版本号；modCount 表示数组实际的版本号。
```

常用的迭代器方法有三个：

+ `hasNext` 还有没有值可以迭代
+ `next`如果有值可以迭代，迭代的值是多少
+ `remove`删除当前迭代的值

**hasNext 源码**

```java
public boolean hasNext() {
  	// cursor 表示下一个元素的大小，size 表示实际存放的元素个数，两者相等时，表示没有元素可以便利了
	  return cursor != size;
}
```

**next源码**

```java
@SuppressWarnings("unchecked")
public E next() {
  	// 检查版本号和预期值是否一致，如果被修改则抛异常
    checkForComodification();
  	// 获取应该返回的元素位置
    int i = cursor;
  	// 超过元素数量，抛出异常
    if (i >= size)
      throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
      throw new ConcurrentModificationException();
  	// 下次迭代时，元素的位置
    cursor = i + 1;
    return (E) elementData[lastRet = i];
}

// 判断是否修改了 elementData 数组
final void checkForComodification() {
    if (modCount != expectedModCount)
      throw new ConcurrentModificationException();
}
```

源码这一块里面，next 做了两件事，第一判断能不能继续迭代，第二找到需要迭代的值，并为下次迭代做准备。

这里面有个很有意思的点，在判断版本号和 i 是否超过元素个数以后，又做了一次是并发修改的判断。新建了一个本地变量，将地址指向 ArrayList 的数组 elementData。此时 elementData 如果修改地址对取值已经没什么影响了。然后判断 i 是否超过数组长度，保证代码正常运行能取值，但是多线程的情况下，取值不一定是对的。

**remove源码**

```java
public void remove() {
    if (lastRet < 0)
      throw new IllegalStateException();
  	// 判断数组是否被修改
    checkForComodification();

    try {
      ArrayList.this.remove(lastRet);
      cursor = lastRet;
      // -1 表示元素已经被删除，防止重复删除
      lastRet = -1;
      // 删除元素会修改modCount，需要给expectedModCount重新复制
      expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
      throw new ConcurrentModificationException();
    }
}
```

这里有两点需要注意的：

+ `lastRet = -1`的操作是为了防止重复删除
+ 删除元素成功，数组的 modCount 会改变，需要同步修改 expectedModCount 的值。

### 6、双向迭代器

除了对`java.util.Iterator`接口类的实现，ArrayList 还有一个双向迭代器的实现，这个类在继承`Iterator`接口实现类的基础上，实现了`java.util.ListIterator`接口。因此又增加了几个逆向迭代方法：

+ `hasPrevious` 前面还有没有值可以迭代，实现代码类似`hasNext` 
+ `nextIndex` 正向迭代时，下一个迭代的位置
+ `previousIndex` 逆向迭代时，下一个迭代的位置
+ `previous` 逆向迭代时，下一个迭代的值，实现类似`next`
+ `set(E e)` 修改当前迭代位置上的元素
+ `add(E e)` 在当前迭代的位置上添加一个元素

双向迭代器的获取方式，一种是调用`listIterator() `方法直接获取，默认起始位置是 0 ；另一种是调用`listIterator(int index)`方法指定迭代器的起始位置

```java
public int nextIndex() {
  	return cursor;
}

public int previousIndex() {
  	return cursor - 1;
}
// 修改指定迭代位置上的值
public void set(E e) {
  if (lastRet < 0)
    throw new IllegalStateException();
  checkForComodification();

  try {
    ArrayList.this.set(lastRet, e);
  } catch (IndexOutOfBoundsException ex) {
    throw new ConcurrentModificationException();
  }
}
// 在迭代的位置上新增一个值
public void add(E e) {
  checkForComodification();
  try {
    int i = cursor;
    ArrayList.this.add(i, e);
    cursor = i + 1;
    lastRet = -1;
    expectedModCount = modCount;
  } catch (IndexOutOfBoundsException ex) {
    throw new ConcurrentModificationException();
  }
}
```

### 7、线程安全

ArrayList 是线程不安全的，是因为 ArrayList 自身的 elementData、size、modConut 在进行各种操作时，都没有加锁。

## 其他问题

### 1、为什么引入 modCount 这个变量？

引入 modCount 变量是为了在遍历集合时，判断是否有并发操作改变数据的标志，在快速失败原理上有使用。

### 2、为什么在添加数据时扩容10，而不是初始化的时候？

节约空间，有时候初始化之后就赋值成别的 list 了

### 3、elementData 为什么使用 transient 修饰？

因为elementData里面不是所有的元素都有数据，因为容量的问题，elementData里面有一些元素是空的，这种是没有必要序列化的。ArrayList的序列化和反序列化依赖writeObject和readObject方法来实现。可以避免序列化空的元素。

### 4、`new ArrayList(0)` 和 `new ArrayList(1)`的实现

` new ArrayList()` 在添加第一个元素时会初始化为 10 个空间，`new ArrayList(0) `在扩容的时候，执行代码`newCapacity = oldCapacity + (oldCapacity >> 1)`，得到的新数组容量也是 0 ，这时候就会根据实际元素的个数扩容 `newCapacity = size + 1`。`new ArrayList(0)`在添加第一个元素后的容量为 1 。

同样的道理，`new ArrayList(1)`在添加 2 个元素时，扩容的新容量为 1，不满足最低的容量需求，这时候按照最低的容量要求扩容。

### 5、ArrayList 存放整型元素，调用remove方法是按照下标移除还是按照元素移除？

当集合添加元素为数字时，添加进去的元素自动装箱成`Integer`类型。我们在删除元素时，如果直接传入参数，会被当作索引处理。

```java
ArrayList list = new ArrayList(0);
list.add(1);
list.add(3);
list.remove(3);
// 输出：class java.lang.Integer
System.out.println(list.get(0).getClass());
// 抛出异常：IndexOutOfBoundsException
list.remove(3);
// 正常删除
list.remove(new Integer(3));
```

### 6、Java 的快速失败和安全失败

*快速失败（fail—fast）*

用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出 `ConcurrentModificationException`

**原理**： 迭代器在遍历时直接访问集合中的内容，并且在遍历过程中会判断 `modCount` 和`expectedmodCount`是否相等。两者相等的情况下才会进行迭代。集合在被遍历期间如果内容发生变化，就会改变 modCount 的值，然后导致两者不相等，会抛出异常，终止遍历

**注意**：通过源码可以看出，虽然做了很多判断，但是并不能保证线程安全。如果正好在判断以后修改了 modCount 的值，并且 elementData 的长度满足条件，那么这次迭代还是能正常运行的。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的 bug。

**场景：**java.util 包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改）。

*安全失败（fail—safe）*

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发 `ConcurrentModificationException`。

**场景：**java.util.concurrent 包下的容器都是安全失败，可以在多线程下并发使用和修改。

### 7、ArrayList 的扩容为什么是 1.5 倍？

**扩容参数为 (1, 2) 之间比较好**。

假设扩容参数为 x，当 `x >= 2` 时，每次扩展的新尺寸必然刚好大于之前分配的总和
$$
c⋅(1+x+x^2+⋯+x^{n−2}) ≤ c⋅x^n
$$
也就是说之前的内存空间都不能进行复用。

```
IF x = 2 :
caps: 1 2 4 8 16 32
---
1
 12
   1234
       12345678
               123456789012345
                              12345678901234567890123456789012

IF x = 1.5 :
caps: 1 2 3 4 6 9 13 19 28
---
1
 12
   123
      1234
123456
      123456789
               1234567890123
                            1234567890123456789
1234567890123456789012345678
```

可以看到，k = 1.5 在几次扩展之后，可以重用之前的内存空间。