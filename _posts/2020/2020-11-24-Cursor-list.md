---
layout:	post
title: "Cursor list"
subtitle: ""
date: 2020-11-24 00:10:00
author: "Yinwhe"
header-style: text
tags:
    - Data Structure
---



# Concept

所谓链表，应该满足两个条件

- 数据存储在一组结构体中。每一个结构体包含有数据以及指向下一个结构体的指针。
- 一个新的结构体可以通过调用malloc而从系统全局内存（global memory）得到，并可以通过free而被释放。

游标法表示链表，本质上是一个结构数组，定义如下

```cpp
struct node{
	elementType value;
	nodePtr next;//int type
}cursor[Maxsize];
```

**Main idea**是将数组的下标作为next存储，cursor应该被初始化为这样的形式：

- 每一个node的next都是当前下标+1，即下一个node，最后一个node的next为0，游标表中用0表示null
- 初始化的cursor有特殊意义，是一个node池，后续的申请或者释放都将源自这个node池。
- 如申请，就将cursor[0].next作为新node，然后更新cursor[0].next=cursor[cursor[0].next].next，具体见 Implement

# Implement

- pre

    ```cpp
    typedef int ptr_to_node;
    typedef ptr_to_node list;
    typedef ptr_to_node position;

    struct node
    {
        element_type     element;
        position         next;
    };

    struct node cursor_space[spacesize];
    ```

- free and malloc

    ```cpp
    static position cursor_malloc(void)
    {
        position p;

        p = cursor_space[0].next;
        cursor_space[0].next = cursor_space[p].next;

        return p;
    }

    static void cursor_free(position p)
    {
        cursor_space[p].next = cursor_space[0].next;
        cursor_space[0].next = p;
    }
    ```

- find

    ```cpp
    position find(element_type X, list L)
    {
        position p;
        
        p = cursor_space[L].next;
        while(p && cursor_space[p].element != X)
            p = cursor_space[p].next;
        
        return p;
    }
    ```

- Insert and delet

    ```cpp
    void delete(element_type X, list L)
    {
    		position p, tmpcell;
        
        p = find(X, L);
        if(!islast(p, L))
        {
            tmpcell = cursor_space[p].next;
            cursor_space[p].next = cursor_space[tmpcell].next;
            cursor_free(tmpcell);
        }
    }

    void insert(element_type X, list L, position P)
    {
        position tmpcell;
        
        tmpcell = cursor_alloc();
        if(tmpcell == 0)
            fatal_error("out of sapce!!!");

        cursor_space[tmpcell].element = X;
        cursor_space[tmpcell].next = cursor_space[P].next;
        cursor_space[P].next = tmpcell;
    }
    ```