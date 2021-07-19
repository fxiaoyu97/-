# int 和 Integer 有什么区别

int 是 Java 提供的 8 种原始数据类型之一。Java 为每个原始类型提供了封装类，Integer 是 Java 为 int 提供的封装类。 int 的默认值为 `0`，而 Integer 的默认值为 `null`，是引用类型，即 Integer 可以区分出未赋值和值为 0 的区别，int 则无法表达出未赋值的情况， Java 中 int 和 Integer 关系是比较微妙的。关系如下：

- int 是基本的数据类型；
- Integer 是 int 的封装类；
- int 和 Integer 都可以表示某一个数值；
- int 和 Integer 不能够互用，因为他们两种不同的数据类型；
