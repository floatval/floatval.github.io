---
title: 泛型动态栈
date: 2020-01-05 16:52:15
categories:
 - 数据结构与算法
tags:
 - 栈
 - 泛型栈
 - 链表
copyright: true
author: 佚
---
# 数组实现的泛型栈
支持各个不同类型的元素来创建栈。模仿 STL 里的栈。

## API 设计
 - bool isEmpty(); 判断栈是否为空

 - void push(T t); 向栈顶添加元素

 - void pop(); 弹出栈底元素

 -  T top(); 获取栈底元素

 - size_t size(); 获取当前栈内元素的个数

<!--more-->
## 设计思路
用数组来模拟栈。设数组头为栈底，数组尾为栈顶。
插入数据时，从数组尾部进行插入，删除数据时从数组尾部进行删除。
### 设计原因
从数组尾部添加删除元素时，不用额外的移动元素。可使得操作的时间复杂度为 O(1)。操作完成后，更新栈顶位置即可。

## 代码实现
```C++
#ifndef STACK_STACK_H
#define STACK_STACK_H


#include <cstdio>
#include <array>

template<typename T>
class stack {
public:
    explicit stack();

    bool isEmpty();

    void push(T t);

    void pop();

    T top();

    size_t size();

private:
    void resize();

private:
    T *m_stack;
    size_t m_size = 0;
    size_t m_capcities=1;
};


#endif //STACK_STACK_H
```
```C++
#include "stack.h"

template<typename T>
stack<T>::stack() {
    m_stack = T(m_capcities);
}

template<typename T>
bool stack<T>::isEmpty() {
    return 0 == m_size;
}

template<typename T>
void stack<T>::push(T t) {
    if (m_size > (m_capcities / 2)) {
        resize();
    }
    memcpy(m_stack + m_size, t, sizeof(T));
}

template<typename T>
void stack<T>::pop() {
    --m_size;
}

template<typename T>
size_t stack<T>::size() {
    return m_size;
}

template<typename T>
T stack<T>::top() {
    return *(m_stack + m_size);
}

/**
 * 动态扩容栈的容量
 * 扩容算法：当前栈内元素 > 1/2 栈的容量时，将栈容量扩容为原来的 2 倍这样可以保证栈使用率不会低于 1/4
 * 然后将扩容前栈内的元素，依次拷贝到新栈内。
 * @tparam T 栈内存放元素的类型
 */
template<typename T>
void stack<T>::resize() {
    m_capcities *= 2;
    size_t length = sizeof(T);
    T *new_stack = T(m_capcities);
    for (int i = 0; i < m_size; ++i) {
        memcpy(new_stack + i * length, m_stack + i * length, length);
    }
    m_stack = new_stack;
}

```
## TODO
移除元素时，判断栈容量，申请小容量栈，将原始栈内数据拷贝到小容量栈中。

# 链表实现的泛型栈
## API 设计
```C++
#ifndef STACK_NODE_STACK_H
#define STACK_NODE_STACK_H


template<class T>
class node_stack {
public:
    node_stack(); // 创建一个新链表栈

    int size(); // 链表栈内的元素个数

    bool isEmpty();// 链表栈是否为空：空返回 false，非空返回 true

    void push(T t); // 向链表栈中添加元素

    T top(); // 获取链表栈的顶部元素

    void pop();// 弹出链表栈的栈顶元素

    //TODO void destory(); 释放链表元素

private:
    typedef struct Node {
        T data;
        Node *next;
    } node;
    int m_size;
    node m_head;
};// 链表节点



#endif //STACK_NODE_STACK_H
```

## 设计思路
用数组来实现栈，有一个弊端就是会造成一定程度上的空间浪费。所以为了规避掉空间上的浪费，采用线性表中的非连续存储结构：栈来实现对应的栈。
实现的栈的特性：添加、删除元素均与现有元素的规模无关。时间复杂度为 O(1)

## 代码实现
```C++
#include "node_stack.h"

template<class T>
node_stack<T>::node_stack() {
    m_size = 0;
    m_head.next = nullptr;
}


/**
 * 链表的头插法，将新节点插入到头节点之后
 * new_node 的指针域指向 head 节点的下一个元素
 * new_node 的数据域，为待添加的元素
 * 将 head 的指针域指向 new_node 完成节点入链
 * @tparam T 待插入数据的类型
 * @param t 待插入数据
 */
template<class T>
void node_stack<T>::push(T t) {
    node *new_node=new node;
    new_node->data=t;
    new_node->next=m_head.next;
    m_head.next=new_node;
    ++m_size;
}

template<class T>
int node_stack<T>::size() {
    return m_size;
}

template<class T>
bool node_stack<T>::isEmpty() {
    return 0 == m_size;
}

template<class T>
T node_stack<T>::top() {
    return m_head.next->data;
}

template<class T>
void node_stack<T>::pop() {
    m_head.next = m_head.next->next;
    --m_size;
}

```

## 弊端
如果频繁的对栈内元素进行删除操作或是销毁栈，此时容易造成内存碎片化。解决思路：使用内存池计数。

# 题外话
最近一直在学前端……难受的一批，好久没更新过了……抽空写一篇水文:-D
