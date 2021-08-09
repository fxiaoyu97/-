## 简单介绍

HashMap 的底层数据结构是：数组 + 链表 + 红黑树。当链表长度大于等于 8 时，链表会转化成红黑树。当红黑树的大小等于 6 时，红黑树会转化成链表。

### 基本特点

+ 允许存放为 null 的 key 和 value
+ 它是线程不安全的，可以加锁保证线程安全。也可以通过Collections#synchronizedMap实现线程安全（每个方法都加锁）
+ 负载因子`(loadFactor)`默认是 0.75，`DEFAULT_LOAD_FACTOR = 0.75f`，扩容的条件为：`需要的数组大小/loadFactor >= 数组容量`
+ 如果有大量数据要储存到 HashMap 中，建议 HashMap 的容量一开始就设置足够的容量，减少扩容的次数。
+ 在迭代的过程中，如果 HashMap 的结构被修改，会快速失败

### 基本属性

```java
// 初始容量 16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

// 最大的容量
static final int MAXIMUM_CAPACITY = 1 << 30;

// 负载因子的默认值
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 桶上的链表长度大于等于 8 时，链表转化红黑树
static final int TREEIFY_THRESHOLD = 8;

// 桶上的红黑树节点个数小于等于 6 时，红黑树转化为链表
static final int UNTREEIFY_THRESHOLD = 6;

// 最小树形化容量阈值：即 当哈希表中的容量 > 该值时，才允许树形化链表 （即 将链表 转换成红黑树）
// 否则，若桶内元素太多时，则直接扩容，而不是树形化
// 为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD
static final int MIN_TREEIFY_CAPACITY = 64;

// 存储数据的数组
transient Node<K,V>[] table;

// 当被调用entrySet时被赋值。通过keySet()方法可以得到map key的集合，通过values方法可以得到map value的集合。
transient Set<Map.Entry<K,V>> entrySet;

// HashMap 存放元素的个数
transient int size;

// 记录 HashMap 被修改的次数，如果在迭代过程中发生变化，会 fail-fast
transient int modCount;

// 扩容的门槛, 当 size 大于 threshold 时会执行 resize 操作
// 如果初始化时，给定数组大小的话，通过 tableSizeFor 方法计算，数组大小永远接近于 2 的幂次方
// 如果是通过 resize 方法扩容，大小 = 数组容量 * 0.75
int threshold;

// 负载因子
final float loadFactor;
```

## 源码解析

### 1、初始化

初始化获取 HashMap 有以下几种方法：

+ 无参初始化
+ 指定容量大小
+ 指定容量大小和负载因子
+ 使用其他 Map 集合作为入参

```java
// initialCapacity 容量大小
// loadFactor 负载因子
public HashMap(int initialCapacity, float loadFactor) {
  	// 判断容量是否为负值
    if (initialCapacity < 0)
      throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
  	// 判断容量是否超过最大值
    if (initialCapacity > MAXIMUM_CAPACITY)
      initialCapacity = MAXIMUM_CAPACITY;
  	// 判断负载因子是否合法
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
      throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
  	// 负载因子赋值
    this.loadFactor = loadFactor;
  	// 计算扩容门槛的大小，通过 tableSizeFor 方法计算，数组大小永远接近于 2 的幂次方
    this.threshold = tableSizeFor(initialCapacity);
}

// 指定容量大小
public HashMap(int initialCapacity) {
  	// 使用默认的负载因子
  	this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 无参初始化，使用默认的负载因子大小
public HashMap() {
 	 // 使用默认的负载因子
 	 this.loadFactor = DEFAULT_LOAD_FACTOR; 
}

// 使用map元素初始化，负载因子为默认大小
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}

// 不仅是初始化调用此方法，putAll 添加元素的时候也会调用此方法
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
  	// 获取元素的个数
    int s = m.size();
  	// 如果元素个数大于 0 时才做处理
    if (s > 0) {
        // 如果数据表是空的
        if (table == null) { // pre-size
          // 使用负载因子计算需要的容量
          float ft = ((float)s / loadFactor) + 1.0F;
          // 如果需要的需要容量不大于最大值，则返回需要的容量值，否则返回HashMap表示的最大容量
          int t = ((ft < (float)MAXIMUM_CAPACITY) ? (int)ft : MAXIMUM_CAPACITY);
          // 如果容量是大于扩容门槛值，重新计算下次扩容的临界值
          if (t > threshold)
              threshold = tableSizeFor(t);
        }
        // 如果数据表不是空的，且元素个数大于扩容门槛值，进行扩容操作
        else if (s > threshold)
            resize();
      	// 新元素添加到集合中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```



### 2、新增

新增元素的大概步骤如下：

1. 根据元素的 key 计算出 hash 值，然后进入到添加元素的函数中
2. 判断数组是否是空的，是空的就进行首次扩容操作，首次扩容的大小为 16
3. 判断根据 hash 计算的位置上是否存在元素
4. 不存在元素时，直接赋值给数组即可
5. 当存在元素时，就要考虑解决 hash 冲突的问题
6. **此时有一个临时变量 e，如果出现 hash 和 key 都相同的节点，旧的节点会赋值给 e，否则 e 为`null`**
7. 此时已经获取到数组上的节点，先判断冲突的点是不是数组上的这个节点，如果是的，则把这个节点赋给临时变量 e，
8. 如果不是数组上的节点，判断数组上的节点的是红黑树节点还是链表节点，方便后面选择不同的方法新增节点
9. 如果是红黑树，调用红黑树新增的方法，返回值赋值给 e
10. 如果是链表，则查找链表中是否存在相同 hash 和 key 的节点，存在返回给 e ，否则在链表末端添加新元素
11. 判断 e 是否为`null`，进而判断是否需要覆盖原来的值。有两种情况会覆盖旧的值：`onlyIfAbsent`为`false` 或者 原来的值为`null`时
12. **e 不为`null`时，不会再继续后续的操作了**
13. 修改`modCount`的值，e 不为`null`时，走不到这一步
14. 判断是否需要扩容，需要的话就进行扩容

**源码**

```java
/**
 * hash 根据key计算出来的hash值
 * key 键
 * value 值
 * onlyIfAbsent 如果hash和key都相等的情况下，是否替换旧值，false是替换，true是不替换
 * evict 表是否在创建模式，如果为false，则表是在创建模式。
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    // 表示存放数据的数组
  	Node<K,V>[] tab; 
    // p 为 i 位置上的 Node 元素
  	Node<K,V> p; 
  	// n 表示数组的长度，i 表示数组索引下标
  	int n, i;
    
    // 如果数组是空的，进行扩容操作
  	if ((tab = table) == null || (n = tab.length) == 0)
        // 扩容
    	n = (tab = resize()).length;
    
    // 计算需要存放数据的位置 (n - 1) & hash ，判断这个位置是否为空，为空时直接添加数据
 	if ((p = tab[i = (n - 1) & hash]) == null)
        // 创建一个新的节点，放到这个位置上，注意这里修改的是 tab 数组，并不是直接修改集合的 table 数组
    	tab[i] = newNode(hash, key, value, null);
    // 如果这个位置不是空时，此时需要解决 Hash 冲突的问题
  	else {
        // e记录跟新元素存在相同hash和key的节点元素，不存在时，e 为 null
    	Node<K,V> e; K k;
        // 如果 key 和 hash 值都相等时，先把此位置上的节点赋给临时变量
    	if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
      		e = p;
        // 判断是不是当前的节点是不是红黑树节点，如果是增使用红黑树的增加方式
    	else if (p instanceof TreeNode)
      		e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 最后就是链表的方式，添加新元素
    	else {
      		for (int binCount = 0; ; ++binCount) {
        		if ((e = p.next) == null) {
                    // 在链表的最后添加元素
          			p.next = newNode(hash, key, value, null);
                    // 判断链表是否需要转化红黑树，长度大于等于 8 时调用转化函数
          			if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            			treeifyBin(tab, hash);
          			break;
        		}
                // 遍历链表的过程中是否存在hash值和key都相同的节点，存在则结束循环
        		if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
          			break;
                // 此时 e = p.next ，保证循环的指针往后移动
        		p = e;
      		}
    	}
        // 判断是否存在相同 hash 和 key 的值
    	if (e != null) { // existing mapping for key
      		V oldValue = e.value;
            // 如果允许替换或者原来位置上的值为null，则替换掉旧值，否则保留原来的值
      		if (!onlyIfAbsent || oldValue == null)
        		e.value = value;
            // 在 HashMap 中这个函数的方法体是空的
      		afterNodeAccess(e);
            // 返回旧的值，注意：此时并没有修改 modCount 的值
      		return oldValue;
    	}
  	}
    // 记录 HashMap 的结构变化
  	++modCount;
    // 如果当前的元素个数达到了扩容的边界，进行扩容操作
  	if (++size > threshold)
    	resize();
    // 在 HashMap 中这个函数的方法体也是空的
  	afterNodeInsertion(evict);
  	return null;
}
```

从源码中我们可以看出一些注意的点：

+ 新增元素时，没有对键和值进行严格的校验，所以键和值都可以为`null`
+ 新增元素是非线程安全的
+ 添加元素时，如果添加 hash 和 key 都相同的元素，这无法引发快速失败的
+ 存在相同 hash 和 key 的节点时，`onlyIfAbsent`为`false` 或者 原来的值为`null`时会覆盖原来的值

### 3、链表的新增

链表的新增比较简单一些，元素直接添加到链表的尾部，需要考虑的就是转化树的问题。

**链表长度大于等于8的时候，一定会转化成链表吗？不会，还要判断 table 数组的长度是否大于等于 64**

```java
/**
* 链表转化红黑树的函数
* tab table数组
* hash 计算链表在数组中的位置
*/
final void treeifyBin(Node<K,V>[] tab, int hash) {
    // n 表示 table 数组的长度
    // index 表示当前元素要插入的元素位置
    // e 存放链表在数组中的头节点
    int n, index; Node<K,V> e;
    // 表格数组为空或者数组容量小于64时，进行扩容操作，而不是转化红黑树
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    // 当数组这个位置不为空时，且头节点不为null时，开始转化成红黑树
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

**为什么是链表大于等于 8 时转化成红黑树呢？**

可以参考源码的注释。链表查询的时间复杂度是`O(n)`，红黑树查询的时间复杂度是 `O(logn)`，在链表不多的时候，使用元素进行遍历比较快，当元素比较多的时候才转化成红黑树，但是红黑树需要的空间是链表的 2 倍，考虑到转化时间和空间损耗，所以我们需要定义出转化的边界值。

参考泊松分布概率甘薯，链表各个长度的命中概率为

```java
     * 0:    0.60653066
     * 1:    0.30326533
     * 2:    0.07581633
     * 3:    0.01263606
     * 4:    0.00157952
     * 5:    0.00015795
     * 6:    0.00001316
     * 7:    0.00000094
     * 8:    0.00000006
     * more: less than 1 in ten million
```

当链表的长度是 8 的时候，出现的概率是 0.00000006，不到千万分之一，所以说正常 情况下，链表的长度不可能到达 8 ，而一旦到达 8 时，肯定是 hash 算法出了问题，所以在这 种情况下，为了让 HashMap 仍然有较高的查询性能，所以让链表转化成红黑树，我们正常写 代码，使用 HashMap 时，几乎不会碰到链表转化成红黑树的情况，毕竟概念只有千万分之 一。

### 4、红黑树的新增

红黑树没吃透！！！

```java
// map是当前的集合
// tab 是存放数据的数组
// h 表示hash值
// k 表示键
// v 表示值
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab, int h, K k, V v) {
    Class<?> kc = null;
    boolean searched = false;
    // 获取根节点
    TreeNode<K,V> root = (parent != null) ? root() : this;
    // 自旋
    for (TreeNode<K,V> p = root;;) {
        int dir, ph; K pk;
        // p 的 hash 值大于 h，说明 p 在 h 的右边
        if ((ph = p.hash) > h)
            dir = -1;
        // p hash 值小于 h，说明 p 在 h 的左边
        else if (ph < h)
            dir = 1;
        // 要放进去 key 在当前树中已经存在了（）
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if ((kc == null &&
                  (kc = comparableClassFor(k)) == null) ||
                 (dir = compareComparables(kc, k, pk)) == 0) {
            if (!searched) {
                TreeNode<K,V> q, ch;
                searched = true;
                if (((ch = p.left) != null &&
                     (q = ch.find(h, k, kc)) != null) ||
                    ((ch = p.right) != null &&
                     (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            dir = tieBreakOrder(k, pk);
        }

        TreeNode<K,V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            Node<K,V> xpn = xp.next;
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            xp.next = x;
            x.parent = x.prev = xp;
            if (xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;
            moveRootToFront(tab, balanceInsertion(root, x));
            return null;
        }
    }
}
```

