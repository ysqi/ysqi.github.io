
---
date: 2016-12-31T11:35:05+08:00
title: "LRU算法"
description: ""
disqus_identifier: 1485833705772513231
slug: "LRU-suan-fa"
source: "https://segmentfault.com/a/1190000000410708"
tags: 
- golang 
- 算法 
categories:
- 编程语言与开发
---

LRU
最近最少使用算法，LRU算法主要用于缓存淘汰。主要目的就是把最近最少使用的数据移除内存，以加载其他数据。

原理
----

    添加元素时，放到链表头
    缓存命中，将元素移动到链表头
    缓存满了之后，将链表尾的元素删除

LRU算法实现
-----------

-   可以用一个双向链表保存数据
-   使用hash实现O(1)的访问

groupcache中LRU算法实现（Go语言）\
<https://github.com/golang/groupcache/blob/master/lru/lru.go>

源码简单注释：

    package lru

    import "container/list"

    // Cache 结构体，定义lru cache 不是线程安全的
    type Cache struct {
        // 数目限制，0是无限制 
        MaxEntries int

        // 删除时, 可以添加可选的回调函数
        OnEvicted func(key Key, value interface{})

        ll    *list.List // 使用链表保存数据
        cache map[interface{}]*list.Element  // map 
    }

    // Key 是任何可以比较的值  http://golang.org/ref/spec#Comparison_operators
    type Key interface{}

    type entry struct {
        key   Key
        value interface{}
    }

    // 创建新的cache 对象
    func New(maxEntries int) *Cache {
        return &Cache{
            MaxEntries: maxEntries,
            ll:         list.New(),
            cache:      make(map[interface{}]*list.Element),
        }
    }

    // 添加新的值到cache里
    func (c *Cache) Add(key Key, value interface{}) {
        if c.cache == nil {
            c.cache = make(map[interface{}]*list.Element)
            c.ll = list.New()
        }
        if ee, ok := c.cache[key]; ok {
                    // 缓存命中移动到链表的头部
            c.ll.MoveToFront(ee)
            ee.Value.(*entry).value = value
            return
        }
            // 添加数据到链表头部
        ele := c.ll.PushFront(&entry{key, value})
        c.cache[key] = ele
        if c.MaxEntries != 0 && c.ll.Len() > c.MaxEntries {
            // 满了删除最后访问的元素
            c.RemoveOldest()
        }
    }

    // 从cache里获取值.
    func (c *Cache) Get(key Key) (value interface{}, ok bool) {
        if c.cache == nil {
            return
        }
        if ele, hit := c.cache[key]; hit {
            // 缓存命中，将命中元素移动到链表头
            c.ll.MoveToFront(ele)
            return ele.Value.(*entry).value, true
        }
        return
    }

    // 删除指定key的元素
    func (c *Cache) Remove(key Key) {
        if c.cache == nil {
            return
        }
        if ele, hit := c.cache[key]; hit {
            c.removeElement(ele)
        }
    }

    // 删除最后访问的元素
    func (c *Cache) RemoveOldest() {
        if c.cache == nil {
            return
        }
        ele := c.ll.Back()
        if ele != nil {
            c.removeElement(ele)
        }
    }

    func (c *Cache) removeElement(e *list.Element) {
        c.ll.Remove(e)
        kv := e.Value.(*entry)
        delete(c.cache, kv.key)
        if c.OnEvicted != nil {
            c.OnEvicted(kv.key, kv.value)
        }
    }

    // cache 缓存数
    func (c *Cache) Len() int {
        if c.cache == nil {
            return 0
        }
        return c.ll.Len()
    }

