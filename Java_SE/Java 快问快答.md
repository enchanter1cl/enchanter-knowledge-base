# Java 快问快答

## String, StringBuffer, StringBuilder

可变性 String 内部 value 是 final 修饰的。所以每次修改 String 的值， 都会产生一个新的对象。 StringBuffer 和 StringBuilder 是可变类，字符串的变更不会产生新的对象。

线程安全 String是不可变类，所以它是线程安全的。 StringBuffer 是线程安全的，因为它每个操作方法都加了 synchronized同步关键字。 StringBuilder不是线程安全的。 所以在多线程环境下对字符串进行操作，应该使用StringBuffer.

性能 String 最低。Buffer 其次。

## HashMap 怎么

要了解Hash 冲突，那首先我们要先了解 Hash 算法和 Hash表。

a. Hash算法，就是把任意长度的输入，通过散列算法，变成固定长度的输出，这个输出结果是散列值。

b. Hash表又叫做“散列表”，它是通过 key 直接访问在内存存储位置的数据结构。在具体实现上，我们通过 hash 函数把 key 映射到表中的某个位置，来获取这个位置的数据，从而加快查找速度。

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230707171819.png) 上图摘自咕泡教育开源文档

c. 所谓 hash 冲突，是由于哈希算法被计算的数据是无限的，而计算后的结果范围有限，所以总会存在不同的数据经过计算后得到的值相同，这就是哈希冲突。

d. 通常解决 hash 冲突的方法有 4 种。HashMap 用的链式寻址法。 链式寻址法，这是一种非常常见的方法，简单理解就是把存在 hash 冲突的 key，以单向链表的方式来存储，比如HashMap 就是采用链式寻址法来实现的。

e.HashMap 在 JDK1.8 版本中，通过链式寻址法 + 红黑树的方式来解决 has h冲突问题，其中红黑树是为了优化Hash 表链表过长导致时间复杂度增加的问题。当链表长度大于 8 并且 hash 表的容量大于 64 的时候，再向链表中添加元素就会触发转化。
