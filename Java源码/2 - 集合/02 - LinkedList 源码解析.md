## 基础介绍

LinkedList 底层数据结构是一个双向链表：

+ 链表的每个节点叫做 Node，在 Node 中，prev属性表示前一个节点的位置，next 属性表示后一个节点的位置
+ first 是双向链表的头节点，它的前一个节点是`null`
+ last 是双向链表的尾节点，它的后一个节点是`null`
+ 当链表中没有数据时，first 和 last 是同一个节点，前后指向都是`null`
+ 因为是个双向链表，只要机器内存足够大，没有大小限制，但是**变量`size`是有大小限制的**

链表中 Node 的源码实现：

```java
private static class Node<E> {
    E item; // 存放的元素
    Node<E> next; // 指向的下一个节点
    Node<E> prev; // 指向的上一个节点

    Node(Node<E> prev, E element, Node<E> next) {
      this.item = element;
      this.next = next;
      this.prev = prev;
    }
}
```

## 源码解析

### 1、添加

+ `add`方法默认是从尾部开始添加，调用内部的`linkLast`
+ `addFirst`方法是从头部开始添加，调用内部的`linkFirst`
+ `add(int index, E element)`可以在 index 位置前面添加一个元素，`index == size`时在最后面添加

**源码**

```java

/**
* 在链表前面添加一个元素
*/
private void linkFirst(E e) {
  	// 暂存头部节点
    final Node<E> f = first;
  	// 创建新节点，新节点的下一个节点指向原来的头节点
    final Node<E> newNode = new Node<>(null, e, f);
  	// 头节点指向新的节点
    first = newNode;
  	// 如果原来的头节点为空，表示是空链表，此时尾节点也指向新的节点
    if (f == null)
      last = newNode;
  	// 不为空的情况下，原来头节点的前置节点指向新节点
    else
      f.prev = newNode;
    size++; // 链表长度+1
    modCount++; // 修改版本号+1
}

/**
* 在链表尾部添加一个元素，操作步骤类似头部添加
*/
void linkLast(E e) {
  	// 暂存尾部节点
    final Node<E> l = last;
  	// 创建新节点，新节点的上一个节点指向原来的尾节点
    final Node<E> newNode = new Node<>(l, e, null);
  	// 尾节点指向新的节点
    last = newNode;
  	// 如果原来的尾节点为空，表示是空链表，此时头节点也指向新的节点
    if (l == null)
      first = newNode;
    else
      l.next = newNode;
    size++; // 链表长度+1
    modCount++; // 修改版本号+1
}
```

Java 的源码中一些注意和学习的点：

1. 新增元素时，没有对值进行严格的校验，所以可以添加`null`值。
2. 新增元素时，需要注意的是节点的`prev`和`next`的赋值
3. 边界的处理，链表为空时，添加数据的处理

### 2、查询

 LinkedList 提供了根据索引查询数据的方法`get(int index)`，通过调用内部的`Node<E> node(int index) `方法获取节点元素，然后返回节点元素的值。

链表的查询相对数组来说比较慢，这里做了一些简单的优化。把链表分为两部分，查询索引在前半部分就从前往后查询，索引在后半部分就从后往前查询。

```java
public E get(int index) {
  	// 校验index的合法性，不合法的index会抛出 IndexOutOfBoundsException 异常
    checkElementIndex(index);
    return node(index).item;
}
/**
* Returns the (non-null) Node at the specified element index.
* 默认修饰符是default，以被这个类本身和同一个包中的类所访问
*/
Node<E> node(int index) {
  // assert isElementIndex(index);
	// 默认 index 的值是合法的
  // 把链表分两半，判断索引在哪部分，从而确定是从前往后还是从后往前查询
  if (index < (size >> 1)) {
    Node<E> x = first;
    for (int i = 0; i < index; i++)
      x = x.next;
    return x;
  } else {
    Node<E> x = last;
    for (int i = size - 1; i > index; i--)
      x = x.prev;
    return x;
  }
}
```

这里有意思，没有直接遍历整个链表，而是做了一次二分操作，链表分为两部分，直接让循环的次数至少减少了一半。为什么只分一次，没有继续二分操作呢？因为链表只能从前往后或者从后往前遍历，分一次已经是极限操作了。

### 3、删除

+ `remove()`：方法内部调用`removeFirst()`方法
+ `remove(int index)`：删除指定位置的元素，先获取节点，然后调用`unlink(Node<E> x) `
+ `remove(Object o)`：从链表中删除指定的值的节点，然后`unlink`方法，可以删除值为`null`的节点
+ `removeFirst()`：删除第一个节点，内部调用`unlinkFirst(Node<E> f)`方法，传参头节点
+ `removeLast()`：删除最后一个节点，内部调用`unlinkLast(Node<E> l)`方法，传参尾节点

```java
E unlink(Node<E> x) {
    // assert x != null;确保x不是null
    final E element = x.item;
  	// 获取x节点的前一个节点和后一个节点
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;
		
    if (prev == null) {
      // 如果前一个节点为null，则表示x是头节点，我们需要头节点变成下一个节点
      first = next;
    } else {
      // 上一个节点的下个节点，指向x的下一个节点
      prev.next = next;
      // x的上一个节点置null
      x.prev = null;
    }
		// 与头节点类似的操作最后需要把x节点的next设置为null
    if (next == null) {
      last = prev;
    } else {
      next.prev = prev;
      x.next = null;
    }
		// 最后把x的值item也设置为null，加速垃圾回收
    x.item = null;
    size--;
    modCount++;
    return element;
}

// 删除最后一个节点
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;
    if (prev == null)
      first = null;
    else
      prev.next = null;
    size--;
    modCount++;
    return element;
}
```

删除节点元素的操作方法传递的参数都是节点元素，节点元素信息是在上级方法中设置好节点信息，比如根据索引设置的时候，先调用`node(int index)`方法获取节点元素信息。

```java
public E remove(int index) {
  	// 检查索引的合法性
    checkElementIndex(index);
    return unlink(node(index));
}
```

### 4、peek、poll、offer 方法

LinkedList 实现了 Deque 接口，Deque 接口继承了 Queue 接口，这就让 LinkedList 实现了一部分 peek、poll、offer 之类的队列操作方法。这些方法跟 add、remove、get 的区别在于链表为空时的处理逻辑不同。

+ `offer`添加元素直接调用`add`方法，跟`add`保持一致
+ **链表为空**获取头节点时，`peek()`返回`null`；`element()`抛出 `NoSuchElementException`异常
+ **链表为空**删除头节点时，`poll()`返回`null`；`remove()`抛出 `NoSuchElementException`异常
+ `push()`方法在头节点添加节点元素

```java
// offer添加元素直接调用add方法，跟add保持一致
public boolean offer(E e) {
  	return add(e);
}
// peek 方法在链表为空时会返回 null 
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
// element 方法在链表为空时，会抛出 NoSuchElementException 异常
public E element() {
  	return getFirst();
}
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
      throw new NoSuchElementException();
    return f.item;
}
```

### 5、迭代器

LinkedList 的迭代器和 ArrayList 的迭代器略有不同，LinkedList 实现了一个双向的迭代器，这个是直接实现了`ListIterator` 接口，迭代器主要包括四个属性值

```java
// 最后一次执行 next() 或者 previous() 方法时返回的节点位置
private Node<E> lastReturned; 
// 下一次迭代的节点
private Node<E> next;
// 下一次迭代的节点位置
private int nextIndex;
// 期望的版本号，判断链表是否修改
private int expectedModCount = modCount;
```

双向迭代器的获取方法只有一种，需要传递一个参数 index ，也就是迭代的起始位置。

```java
public ListIterator<E> listIterator(int index) {
    checkPositionIndex(index);
    return new ListItr(index);
}
```

主要方法包括：

+ `hasNext` 后面还有没有值可以迭代
+ `hasPrevious` 前面还有没有值可以迭代，实现代码类似`hasNext` 
+ `nextIndex` 正向迭代时，下一个迭代的位置
+ `previousIndex` 逆向迭代时，下一个迭代的位置
+ `next` 正向迭代时，下一个迭代的值
+ `previous` 逆向迭代时，下一个迭代的值，实现类似`next`
+ `set(E e)` 修改当前迭代位置上的元素
+ `add(E e)` 在当前迭代的位置上添加一个元素

**正向迭代源码**

```java
// 判断是否还有元素可以迭代
public boolean hasNext() {
  return nextIndex < size;
}

public E next() {
  // 检查链表是否修改
  checkForComodification();
  if (!hasNext())
    throw new NoSuchElementException();
  // 设置需要返回的节点信息
  lastReturned = next;
  // 为下一次迭代做准备
  next = next.next;
  nextIndex++;
  return lastReturned.item;
}
```

**逆向迭代源码**

这里比较有意思，因为在获取迭代器的时候需要传参，这个参数大小可以是 size ，当 index 的值等于 size 时，next的值就是`null`，这时候在调用 `next()` 方法的时候会抛出异常。但是在调用`previous`时就需要做一些特殊处理。

```java
// 判断需要迭代的节点位置是否大于 0
public boolean hasPrevious() {
  return nextIndex > 0;
}

public E previous() {
  // 判断链表是否被修改
  checkForComodification();
  if (!hasPrevious())
    throw new NoSuchElementException();
	// 如果 next 为 null 时，说明创建迭代器时传的参数值等于size，前一个节点正好是链表最后一个节点
  lastReturned = next = (next == null) ? last : next.prev;
  // 修改索引位置
  nextIndex--;
  return lastReturned.item;
}
```

**迭代器删除**

因为是双向迭代操作，这个删除就没有那么简单了。删除的时候要考虑两种情况，一种是正向迭代，一种就是逆向迭代。

```java
public void remove() {
    checkForComodification();
    if (lastReturned == null)
      throw new IllegalStateException();
		// 暂存要删除节点的下一个节点
    Node<E> lastNext = lastReturned.next;
  	// 删除节点
    unlink(lastReturned);
  	// 如果下个节点等于要删除的节点，这种情况一般出现在逆向迭代的过程中
  	// lastReturned = next = (next == null) ? last : next.prev;
    if (next == lastReturned)
      next = lastNext;
    else
      // 正向迭代的时候，next 已经指向了下一个节点，并且执行nextIndex++，此时只要修正 nextIndex 的值
      nextIndex--;
    lastReturned = null;
    expectedModCount++;
}
```

**单向迭代器(descendingIterator)**

LinkedList 在JDK6的时候通过**实现`Iterator`接口，实现了一个逆向的迭代器，它是从后往前迭代元素的。**这个迭代器本质上还是`ListIterator`迭代器，只不过又做了一层封装。

```java
public Iterator<E> descendingIterator() {
  	return new DescendingIterator();
}

private class DescendingIterator implements Iterator<E> {
  	// 获取链表的 ListIterator 迭代器，传参为链表的长度
    private final ListItr itr = new ListItr(size());
    public boolean hasNext() {
      return itr.hasPrevious();
    }
    public E next() {
      return itr.previous();
    }
    public void remove() {
      itr.remove();
    }
}
```

