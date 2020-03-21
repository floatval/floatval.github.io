---
title: 泛型链式队列的实现
date: 2020-01-06 20:28:31
categories:
 - 数据结构与算法
tags:
 - 队列
 - 泛型队列
 - 链表
copyright: true
author: 佚
---

# 泛型链式队列
采用链式队列的原因跟采用链式栈的原因一致：为了 **避免使用数组来进行实现时，动态扩容时造成的不必要的空间浪费** 。

## API 设计

```C++
#ifndef QUEUE_QUEUE_LINK_H
#define QUEUE_QUEUE_LINK_H

template <class T>
class queue_link {
public:
    queue_link(); // 创建队列
    
    ~queue_link(); // 析构队列（ps：未实现）
    
    void enqueue(T t);// 元素入队
    
    void dequeue();// 元素出队
    
    unsigned int size();// 获取当前队列中元素的个数
    
    T front();// 获取队列中首部元素
    
    T back();// 获取队列中尾部元素
    
    bool isEmpty();// 判断队列是否为空

private:

typedef struct Node{
    T data;
    Node* next;
} node;

private:
    node m_head;// 头节点
    node** m_head_pointer; // 头指针
    node** m_tail_pointer; // 尾指针
    unsigned int m_size;
};
```

<!--more-->

## 设计思路
 - 链式队列采用 **头删尾插法**
     - 原因：1. 若采用 **头插尾删** 法时，单向链表无法实现取上一个元素。即使保留有尾指针，在每次删除过元素之后，
     均需要从头节点开始遍历到尾节点，然后重新为尾指针赋值。这样效率太低
    
     - 原因：2. 采用 **头删尾插法**，只需要维护一个指向链表第一个元素的头指针，一个指向链表尾部元素的尾指针，
     和一个头节点即可。这样因为有头节点的存在，可以确定链表的第一个元素位置，每次从头部删除元素后，可以很方便的
     更新头指针。尾部插入数据，可以很好的规避单向链表无法逆序访问的缺点。
   
     - 注意：当链表中只没有元素时，添加数据时需要同时更新头指针和尾指针。
     - 当链表中有 **1个** 元素时，删除元素时，需要同时更新头指针和尾指针

## 代码实现

```C++
#include "queue_link.h"

template<class T>
queue_link<T>::queue_link() {
    m_head_pointer=new node*;
    m_tail_pointer=new node*;
    m_head.next = nullptr;
    *m_head_pointer = &m_head;
    *m_tail_pointer = &m_head;
    m_size = 0;
}

template<class T>
queue_link<T>::~queue_link() = default;

/**
 * 从链表的尾部添加数据，实现队列中元素的入队
 * 1. 将元素添到 new_node 的数据域,并将 new_node 的指针域指向尾指针的指针域的指向
 * 2. 更新尾指针指向元素的指针域，使其指向 new_node
 * 3. 更新尾指针指向 PS：如果链表中只有一个元素的话，同时需要更新头指针
 * @tparam T 队列中的元素的类型
 * @param t 添加到队列中的数据
 */
template<class T>
void queue_link<T>::enqueue(T t) {
    std::cout<<"元素开始入站"<<std::endl;
    node *new_node = new node;
    new_node->data = t;
    new_node->next = nullptr;
    *m_tail_pointer = new_node;
    // 第一次添加元素
    if (nullptr == m_head.next) {
        // 链表中只有一个元素
        *m_head_pointer = new_node;
        m_head.next = *m_head_pointer;
    }
    // 节点入链表
    *m_tail_pointer = new_node;
    ++m_size;
}

/**
 * 将元素从链表头部移除，模拟元素的出队
 * 1. 更新头指针，将头指针指向待删除元素的下一个元素(如果只有一个元素的话，同时要更新尾指针）
 * 2. 通过头节点来释放待删除元素（防止内存泄漏）
 * 3. 更新头节点的指针域指向，使得指针指向当头节点
 * @tparam T 队列中的元素类型
 */
template<class T>
void queue_link<T>::dequeue() {
    if (*m_head_pointer == *m_tail_pointer) {
        // 链表中只有一个元素
        delete *m_head_pointer;
        (*m_head_pointer)->next = nullptr;
        m_head.next = nullptr;
        *m_head_pointer = *m_tail_pointer = &m_head;
        --m_size;
    } else {
        *m_head_pointer = (*m_head_pointer)->next;
        delete m_head.next;
        m_head.next->next = nullptr;
        m_head.next = *m_head_pointer;
        --m_size;
    }
}

template<class T>
T queue_link<T>::front() {
    return (*m_head_pointer)->data;
}

template<class T>
T queue_link<T>::back() {
       return (*m_tail_pointer)->data;
}

template<class T>
unsigned int queue_link<T>::size() {
    return m_size;
}

template<class T>
bool queue_link<T>::isEmpty() {
    return 0==m_size;
}
```
