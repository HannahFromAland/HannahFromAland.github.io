---
title: 【Data Structure】HashMap
date: 2021-06-16 18:23:32
tags: Data Structure

---

试图通过整理解决目前遇到过的HashMap全部问题。

<!-- more -->

## 底层数据结构

HashMap是由数组+链表+JDK1.8中增加的红黑树（当链表长度大于8时转换为红黑树）实现的。

> 在Java中，数据结构的物理存储只有数组好链表两种方式且各有优势，其中数组存储区域连续，因此查找快，由于增删要改动整个数组，因此增删慢；而链表反之。将数组和链表结合在一起，构成既满足查找快也满足增删块的基本数据结构，就构成了HashMap.

![image.png](https://i.loli.net/2021/06/16/hvoyXqMSO95eZWd.png)

- HashMap中最基本的单元是Entry。
- 数组中Entry的“寻位”使用的是HashCode，通过对Key值进行Hash得到数据在数组中存放的下标获得位置。

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;    //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next;   //链表的下一个node

        Node(int hash, K key, V value, Node<K,V> next) { ... }
        public final K getKey(){ ... }
        public final V getValue() { ... }
        public final String toString() { ... }
        public final int hashCode() { ... }
        public final V setValue(V newValue) { ... }
        public final boolean equals(Object o) { ... }
}
```

- Node[] table，在逻辑结构中表现为用于存放Node的数组。Node为实现Map中Entry接口的键值对。

在HashMap中还定义了几个比较重要的字段（分别对应相关的属性），具体源码如下：

```java
public class HashMap<K,V> extends AbstractMap<K,V> 
	implements Map<K,V>, Cloneable, Serializable {

	 /**
     * HashMap的默认初始容量大小 16
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * HashMap的最大容量 2的30次方
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 负载因子，代表了table的填充度有多少，默认是0.75。
     * 当数组中的数据大于总长度的0.75倍时,HashMap会自动扩容
     * 默认扩容到原长度的2倍。为什么是2倍，而不是1.5倍，或是3倍?
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 默认阈值，当桶(bucket)上的链表长度大于这个值时会转成红黑树
     * 在jdk1.7中链表就是普通的单向链表，因而导致链表很长，平均效率降低
     * jdk1.8对此做了优化，默认当链表长度大于8时转化为红黑树
     */
    static final int TREEIFY_THRESHOLD = 8;
    
    /**
     * 和上一个的阈值相对的阈值，当桶(bucket)上的链表长度小于这个值时红黑树退化成链表
     */
    static final int UNTREEIFY_THRESHOLD = 6;
    
    /**
     * 用于快速失败
     * HashMap非线程安全，
     * 在对HashMap进行迭代时，如果期间其他线程的参与导致HashMap的结构发生变化	  * 了（比如put，remove等操作），需要抛出异常					ConcurrentModificationException
     */
    transient int modCount;
    int size; //实际存在的键值对数量
    

}
```

- 其中，初始容量和负载因子可以通过设置initialCapacity和loadFactor的值进行修改。例如在内存空间较多，时间效率要求较高的特殊情况下，可以降低负载因子的值；反之可增加负载因子的值。

## 解决冲突

HashMap通过Hash算法来解决冲突。Java的HashMap采用了链地址法，即上图中链表和数组的结合。 

当有新的键值对需要被放入HashMap时，采用Hash算法对Key进行运算后可以得到数组下标，之后将键值对放在对应下标元素的链表上即完成HashMap的存储功能。

- 当两个Key的Hash值相同时，在table数组中会定位到相同的位置，此时即发生了Hash碰撞。解决碰撞的方法除链地址法以外，还可以通过**开放地址法**实现（即当某个槽位已满时，通过其他放大继续查找下一个可以使用的槽位。e.g.线性探测法，二次散列探测法）
- Java在HashMap中形成但链表的核心设计在于，总是将新添加的Entry对象放入table数组的索引处。具体添加方式要依次进行以下判断：
  - table是否为空
  - 该位置是否已经存在Key
  - 对应的Key是否为treenode（已经变为红黑树的存储结构）
  - 链表长度是否大于8（是否需要变为红黑树的存储结构）
- 在没有出现Hash冲突时，HashMap查找元素很快；但出现单链表之后，需要在同一个bucket中对Entry链进行遍历才能找到需要的元素（最坏情况是该Entry是最早放入bucket中的元素）。

## Reference

[HashMap的底层实现](https://lushunjian.github.io/blog/2019/01/02/HashMap%E7%9A%84%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0/)
[Java 8系列之重新认识HashMap](https://tech.meituan.com/2016/06/24/java-hashmap.html)
