---
layout: post
title: HashMap的resize方法分析
subtitle: 
author: Shiayanga
categories: [HashMap]
banner:
tags: [Java,HashMap]
sidebar: []
---

## 使用场景

+ 场景一
  + HashMap初始化后第一次put键值

+ 场景二
  + HashMap中的元素数量达到阈值

## 分析

+ 场景一：初始化后第一次put

  + 未指定容量

    容量为默认的16；加载因子为默认的0.75；阈值为0

  + 指定容量为16

    容量为16；加载因子为0.75；阈值为16（由 `tableSizeFor(initialCapacity)` 计算所得）

+ 场景二

  + 元素数量达到阈值，触发扩容机制




~~~ java
// 默认初始容量为 16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
// 最大容量 1073741824
static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认加载因子 0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 在首次使用时初始化，根据需要调整大小
transient Node<K,V>[] table;

final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    // HashMap 初始化完成后，table 为 null
    int oldCap = (oldTab == null) ? 0 : oldTab.length;

    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {	
        // 场景二：扩容
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 当容量达到最大容量，将阈值置为Integer的最大值，并返回原有的节点
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY){
            // 扩容后依然未达到最大值，且原有容量大于等于默认初始容量时，将阈值置为当前阈值的两倍
            newThr = oldThr << 1;
        }           
    }else if (oldThr > 0){
      // 场景一：初始化HashMap
      newCap = oldThr;  
    }else {
        // 场景一：初始化HashMap
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        // 如果阈值依然为0，设置新的阈值
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
~~~

