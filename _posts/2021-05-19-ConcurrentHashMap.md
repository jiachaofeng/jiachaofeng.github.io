---
title: ConcurrentHashMap
date: 2021-05-19 00:00:00 +0800
author: CharlesJia
categories: [Java]
tags: [Interview]
---

## ConcurrentHashMap

### **一、简单回顾ConcurrentHashMap在jdk1.7中的设计**

先简单看下ConcurrentHashMap类在jdk1.7中的设计，其基本结构如图所示：

![764863-20160620202714522-1795796503](https://cdn.jsdelivr.net/gh/jiachaofeng/images/static/764863-20160620202714522-1795796503.png)

每一个segment都是一个HashEntry<K,V>[] table， table中的每一个元素本质上都是一个HashEntry的单向队列。比如table[3]为首节点，table[3]->next为节点1，之后为节点2，依次类推。

```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V>
        implements ConcurrentMap<K, V>, Serializable {

    // 将整个hashmap分成几个小的map，每个segment都是一个锁；与hashtable相比，这么设计的目的是对于put, remove等操作，可以减少并发冲突，对
    // 不属于同一个片段的节点可以并发操作，大大提高了性能
    final Segment<K,V>[] segments;

    // 本质上Segment类就是一个小的hashmap，里面table数组存储了各个节点的数据，继承了ReentrantLock, 可以作为互拆锁使用
    static final class Segment<K,V> extends ReentrantLock implements Serializable {
        transient volatile HashEntry<K,V>[] table;
        transient int count;
    }

    // 基本节点，存储Key， Value值
    static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile V value;
        volatile HashEntry<K,V> next;
    }
}
```

