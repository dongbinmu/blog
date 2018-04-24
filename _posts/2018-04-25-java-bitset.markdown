---
layout: post
title: BitSet
date: 2018-04-24 10:20:20 +0300
description:
img:
tags: [Java]
---


#### 简述

通过位来节约空间。

比如在0-64中，有30 个随机数，求出没有在出现的数。
```java
        Random random = new Random();
        BitSet bitSet = new BitSet(64);
        for(int i=0;i<30;i++){
            int tmp = random.nextInt(64);
            bitSet.set(tmp);
            System.out.print(tmp+"\t");
        }
        System.out.println();
        for(int i=1;i<=64;i++){
            if(!bitSet.get(i)){
                System.out.print(i+"\t");
            }
        }
```

就是把值set到对应的二进制位上，将初始的0设置为1

bitset内部维护了一个long[] 数组，因为是long，所以对应的没有就是64位，即index=0 ---[0,63]
index=1 --[64,127]...

```java
    public void set(int bitIndex) {
        //bitIndex是否合法
        if (bitIndex < 0)
            throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);

        //计算当前值存储在具体数组哪一个index,计算的时候>>6 因为long的长度为64位
        int wordIndex = wordIndex(bitIndex);

        //判断是否需要扩容，因为有可能这个index还没有创建
        expandTo(wordIndex);
        /**
        * 先 1L << bitindex ,注因为L 所以bitIndex会先求模，在向左移位
        * 与words[wordIndex]做位或运算并赋值给words[wordIndex]，其实就是将左移的位置0变为1
        */
        words[wordIndex] |= (1L << bitIndex); // Restores invariants

        checkInvariants();
    }
```

其实这里暴露出了一个问题，当我的bitIndex 和其他的元素差异大的时候，可能一个words[index]就绑定了一个值而已，显然是连续的数是比较好的。

#### 应用

在实际中我们更多的是用字符串，并不是数字。则我们可以运用字符串的hashcode值，在hashmap的基础里我们知道，再好的哈希函数都会产生哈希碰撞，就是2个不相同的字符串得到了一个hashcode,在bitset 里 就会存在错误。（这里的错我我们是必须容忍的，只是可以尽量调整出错的几率）

 着色
就是计算出一个字符串的hashcode后，字符串做一个固定改变，看看hashcode是否相同，如果相同 我们得到结果是 这个字符串极有可能存在（存在的概率更高而已）

```java
        BitSet bitSet = new BitSet(Integer.MAX_VALUE);
        String str = "";

        if(bitSet.get(str.hashCode()) && bitSet.get(("xxx-"+str).hashCode())){
            System.out.println("存在");
        }

        //在插入的时候要set 2个值 一个是str的hashcode 另一个是"xxx-"+str 的hashcode

        //第二种是在用一个bitset来存新的hashcode 及"xxx-"+str的hashcode
```

在这个基础上 就有了Bloom Filter

这个的原理是 Bloom Filter 是一个有m位的数组，初始为零，
现在一个集合，当一个元素和k个哈希函数求值，每一个都和数组作位或，若果已经为1的就不在设置。
例子 元素1 在m数组的 第一位和 第三位 等等位为1

谷歌集成的jar

```
    <dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>22.0</version>
		</dependency>
```

```java
    Charset charset = Charset.forName("UTF-8");
    //500 最大个数 0.01 最大容忍错误
	BloomFilter<String> bloomFilter =
	BloomFilter.create(Funnels.stringFunnel(charset),500,0.01);
	bloomFilter.put("dongbin");
	bloomFilter.mightContain("dongbin");

```