---
layout: post
title: ArrayList
date: 2018-04-19 00:20:20 +0300
description:
img:
tags: [Java,集合]
---

#### 构造方法

```java
    //默认构造方法给elementData数组大小为10
    public ArrayList() {
      this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }


    //给定初始大小为initialCapacity的数组
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
          //检查给定的大小是否大于0，大于0则创建一个initialCapacity大小的Object数组
          this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
          //如果传入的是0，则会建一个空数组
          this.elementData = EMPTY_ELEMENTDATA;
        } else {
          //小于0 抛异常
          throw new IllegalArgumentException("Illegal Capacity: "+initialCapacity);
        }
    }

    //初始化一个Collection集合
    public ArrayList(Collection<? extends E> c) {
        //当前数组指向传入集合的数组应用
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

#### add(E)
```java
    //在数组后面添加一个元素
    public boolean add(E e) {
        //校验 当前的元素能加到数组
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //加入数组
        elementData[size++] = e;
        return true;
    }

    private void ensureCapacityInternal(int minCapacity) {
        //如果elementData为空，创建一个10和minCapacity最大值得数组
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // 超出当前数组，执行扩容操作
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        //新的数组长度为原先的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果新的数组长度还小于minCapacity，则minCapacity为新的数组长度
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果新的数组长度大于最大的数组长度，
        //则newCapacity = minCapacity > MAX_ARRAY_SIZE) ?Integer.MAX_VALUE :MAX_ARRAY_SIZE
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    //在指定的位置添加元素
    public void add(int index, E element) {
        //校验index是否在0-size之间
        rangeCheckForAdd(index);
        //校验 保证当前的元素能加到数组
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //把index后的元素都向后移一位
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```

#### remove(E)

```java
    //移除元素
    public boolean remove(Object o) {
        //如果元素为NULL 找到数组中的NULL的删除
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            /**
            *遍历数组，查找元素o，将其删除第一个
            *注意list为 3，2，4，4，2，3，6，5 删除3 不会把所有值为3的删除只会删除第一个
            */
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    private void fastRemove(int index) {
        modCount++;
        //index后的元素向前移一位
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

    remove(int index);//多一个数组范围检查
```

#### get(E)

```java
public E get(int index) {
        //检查数组范围
        rangeCheck(index);
        return elementData(index);
    }
```

#### iterator迭代器

```java
private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;
        //游标是否等于数组的长度
        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            //检查遍历的过程中有没有add和remove
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            //检查遍历的过程中有没有add和remove
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                //迭代移除元素不报错的原因，重新更新了expectedModCount值
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

### 排序

```java
    //排序过程中modCount会变，在排序和iterator不能同时进行
    public void sort(Comparator<? super E> c) {
        final int expectedModCount = modCount;
        //借助的Arrays的排序功能
        Arrays.sort((E[]) elementData, 0, size, c);
        //排序的时候 不能add 或remove
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
```

#### 其他

- ArrayList基于数组实现，没有容量限制
- 在插入元素的时候需要扩容，但是在删除的时候没有减少数组的容量，ArrayList提供了trimToSize
 ```java
 public void trimToSize() {
        //注意modCount也在改变
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
 ```
 - ArrayList不是线程安全的