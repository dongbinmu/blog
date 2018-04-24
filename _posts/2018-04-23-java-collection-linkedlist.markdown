---
layout: post
title: LinkedList
date: 2018-04-23 10:20:20 +0300
description:
img:
tags: [Java,集合]
---

#### 实现方式

```java

    //元素个数
    transient int size = 0;

    //头结点
    transient Node<E> first;

    //尾结点
    transient Node<E> last;

    private static class Node<E> {
        E item;//链表结点所存储数据
        Node<E> next;//当前链表结点的下一个结点
        Node<E> prev;//当前链表结点的上一个结点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

#### 构造方法

```java
    //空的构造方法
    public LinkedList() {
    }

    //初始化的添加一个collection集合
    public LinkedList(Collection<? extends E> c) {
        //调用默认的构造方法
        this();
        //添加参数
        addAll(c);
    }

    //添加集合c到list
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    //添加集合c到list的指定index位置
    public boolean addAll(int index, Collection<? extends E> c) {
        //检查index是否合法
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        /**
        * 如果index等于list的长度
        * 当前结点为空，则获取当前结点为空 前驱指向last结点
        * 否则查找index所在的位置结点，返回当前结点赋值到succ，前驱指向当前结点的上一个结点
        */
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        /**
        * 新生成一个链表
        */
        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            //前驱为空 则first为当前结点,否则前驱的next为现在的结点
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        //如果succ为null，直接把last结点指向最后的结点(相当于在末尾插入元素)
        if (succ == null) {
            last = pred;
        } else {
            //让succ和新的链表结合起来
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }

    /**
    * 查找index位置的node 如果index小于size的一半，从first结点开始查找
    * 否则从last结点开始查找
    */
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

#### add(E)

```java
    //插入结点默认在末尾添加
    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

#### remove(E)

```java
    // 删除结点
    public boolean remove(Object o) {
        //如果结点为null，删除结点第一个为null的结点
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            //删除第一个结点值为o的
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }

    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;
        //如果当前结点是第一个结点，让first指向next
        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            //方便GC
            x.prev = null;
        }

        //如果结点是尾节点，让last指向上一个结点
        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            //方便GC
            x.next = null;
        }

        //方便GC
        x.item = null;
        size--;
        modCount++;
        return element;
    }

```

#### get(E)

```java
    public E get(int index) {
        //检查index是否在范围里
        checkElementIndex(index);
        //查找结点，前面有分析
        return node(index).item;
    }
```

#### 其他

- LinkedList基于双向链表机制实现
- LinkedList在1.8中是用来一个头结点first和尾节点来实现，大大提交了效率


