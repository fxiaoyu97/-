# ArrayList 与 Vector 区别

- 同步性：Vector 是线程安全的，也就是说是同步的 ，而 ArrayList 是线程不安全的，不是同步的。
- 数据增长：当需要增长时，Vector 默认增长为原来一倍 ，而 ArrayList 却是原来的 50% ，这样 ArrayList 就有利于节约内存空间。

说明：如果涉及到堆栈，队列等操作，应该考虑用 Vector，如果需要快速随机访问元素，应该使用 ArrayList