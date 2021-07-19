# HashSet 和 HashMap 区别

| HashMap                                      | HashSet                                                      |
| -------------------------------------------- | ------------------------------------------------------------ |
| HashMap 实现了 Map 接口                      | HashSet 实现了 Set 接口                                      |
| HashMap 储存键值对                           | HashSet 仅仅存储对象                                         |
| 使用 put() 方法将元素放入 map 中             | 使用 add() 方法将元素放入 set 中                             |
| HashMap 中使用键对象来计算 hashcode 值       | HashSet 使用成员对象来计算 hashcode 值，对于两个对象来说 hashcode 可能相同，所以 equals() 方法用来判断对象的相等性，如果两个对象不同的话，那么返回 false |
| HashMap 比较快，因为是使用唯一的键来获取对象 | HashSet 较 HashMap 来说比较慢                                |

