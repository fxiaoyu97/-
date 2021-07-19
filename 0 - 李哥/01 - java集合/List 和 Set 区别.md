# List 和 Set 区别

- List, Set 都是继承自 Collection 接口
- List 特点：元素有放入顺序，元素可重复。Set 特点：元素无放入顺序，元素不可重复（注意：元素虽然无放入顺序，但是元素在 set 中的位置是有该元素的 HashCode 决定的，其位置其实是固定的）
- List 接口有三个实现类：LinkedList，ArrayList，Vector。Set 接口有两个实现类：HashSet(底层由 HashMap 实现)，LinkedHashSet