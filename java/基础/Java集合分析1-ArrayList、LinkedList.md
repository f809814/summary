<h2 align="center">Java集合分析1-ArrayList、LinkedList</h2>

#### 1.ArrayList

- 底层通过数组保存数据，无参构造器构造一个空数组，第一次add元素时才扩容，默认容量10，每次扩容为原来数组大小的1.5倍
- 自定义实现序列化和反序列化（elementData数组为transient），这样可以只序列化存有元素的那一部分数据（大小size），而没有保存元素的数组则不需要序列化
- 使用iterator遍历可能会引发多线程异常，获取iterator时有一个modCount，遍历过程中如果执行了add或remove等会改变modCount的方法，会抛出ConcurrentModificationException异常，set和get不会改变modCount

---

#### 2.LinkedList

- 底层通过双向链表保存元素
- 常用操作方法
  - push == addFirst，pop == removeFirst
  - offer == addLast，poll == removeFirst
  - add == addLast, remove == removeFirst
- 通过下标获取某个node的时候，会根据index处于前半段还是后半段来确定查找方向，以提升查询效率

- 通过下标访问元素Node<E> node(int index) ，会根据下标在前半部分还是后半部分来决定从哪个方向开始查找
- 注意push