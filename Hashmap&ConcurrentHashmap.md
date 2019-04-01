<h1>Hashmap & ConcurentHashMap</h1>
Hashmap是java中最常用的集合类，并且jdk在hashmap的基础上将value置空实现了set结构，hashmap也是面试经常考察的点。本文是简单描述hashmap和concurrenthashmap的结构，详细请查看**<a href="http://www.importnew.com/28263.html">Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析</a>**

## 基础知识

### 哈希算法
哈希算法是一种从任意对象或文件中创造出小的数字“指纹”的方法，可用于加密保护数据。一个好的哈希算法的特点有：
- **正向快速**：可以在有限的时间与资源内计算出哈希值
- **逆向困难**：给点哈希值，在有限的时间内很难逆推出计算哈希之前的值
- **输入敏感**：原数据稍微有一点改动，产生出来的哈希值区别非常大
- **冲突避免**：很难找到内容不同的对象，使得它们在相同的哈希算法计算出来的哈希值一样

### 红黑树
红黑树是一种平衡二叉树，其二分查找和排序性能都很强，但插入比较麻烦，涉及二叉树的平衡。特性：
- 节点不是红色就是黑色
- 根节点必定为黑色
- 叶子节点都是黑色的值为null的节点
- 每个红色节点的子节点必定是黑色的
- 从任意一个节点到其每个叶子节点的所有路径都包含相同数目的黑色节点

由于这些特征保证了红黑树必定是平衡二叉树

## HashMap
### Java7的HashMap
<img src="https://javadoop.com/blogimages/map/1.png"></img>

- HashMap内部是一个数组，每个数组的元素是一个单向链表
- capacity: 当前数组的容量，始终保持为2的n次方，扩容的方式是将capacity左移1位，即翻倍
- loadFactor: 负载因子，默认为0.75。当当前数组内的元素为容量的0.75时，触发扩容
- 插入数据时，使用哈希算法计算出插入的位置，如果插入的位置为空，直接插入。如果插入的位置已经有数据，即发生了哈希碰撞，新数据放在链表头部，旧数据跟在新数据后面
- 扩容需要遍历并且重新计算哈希并调整位置，成本高
- 优化：定义hashmap时指大小，尽量避免扩容。hashcode算法尽量避免碰撞

### Java7的ConcurrentHashMap
<img src="https://javadoop.com/blogimages/map/3.png"></img>

- 简单来说，ConcurentHashMap在HashMap的基础上引入Segment数组，Segment是一个继承了ReentrantLock的数据结构，以此进行加锁，保证每个Segment是线程安全的，实现分段锁
- concurrencyLevel: 并发数。默认为16，表示Segment数组的容量为16，最多支持16个线程的的读写，**一旦初始化完成，不可修改**
- Segment的下一层HashEntry可以看作是原生的HashMap

put操作：
- 进行第一次key的hash操作，定位插入的Segment的位置
- 通过Segment继承的ReetrantLock的tryLock()方法尝试锁住Segment
- 若Segment未初始化，通过CAS进行Segment的初始化并赋值
- 确定Segment之后进行第二次hash操作，找到对应的HashEntry位置，并操作普通HashMap一样写入

size的计算：先使用不加锁的方式尝试3次计算size值，如果三次一致，则表示size是正确的。如果不一致，则进行加锁计算

### Java8的HashMap
**Java8的HashMap在Java7的HashMap的基础上引入了红黑树，以减少数据量大时的检索开销，但链表的数据量大于8时，链表会自动转为红黑树进行数据存储**
<img src="https://javadoop.com/blogimages/map/2.png"></img>

### Java8的ConcurrentHashMap
<img src="https://javadoop.com/blogimages/map/4.png"></img>

Java8的ConcurrentHashMap在普通HashMap的基础上增加了很多的CAS操作，以及在插入的使用使用synchronized关键字上锁

- **key和value不允许为null**
- segment有所保留，但已经简化，仅仅是为了兼容旧版本
- sizeCtl属性： 负数代表HashMap正在扩容或者初始化，-1代表初始化，-N代表有N-1个线程正在执行扩容操作。正数或0代表还没被初始化，这个值代表初始化或者下一次进行扩容的大小
