---
layout: post
title: 求一个数的N次方(n 为整数)
date: 2018-04-10 00:20:20 +0300
description: 求一个数的N次方(n 为整数)
img: i-rest.jpg # Add image post (optional)
tags: [算法]
---

给定一个double类型的浮点数base和int类型的整数n。求base的n次方。

分析：
首先这里不能按照传统的方法去计算,先求整，再循环n个base相乘，最后根据n的正负取值(网上很多都是这样写的)。这并不是真正的最好的算法，很久之前再一本书上看见了一个思路，用移位的方法来计算。

假如 要算X的8次方，可以先算X的4次方；要算X的4次方，可以先算X的2次。通过这个例子可以感觉到和二进制有关，就可以用移位运算。

X的5次方，5的二进制是101，其实结果就是 （1乘以X的4次方） 乘以 （0 乘以X的2次方）
乘以 （1 乘以 X）。

这样就简化了很多循环次数，再n很大的时候能提高效率。

代码实现

```java
public double power(double base, int n) {
        double result = 1.0;        
        int tmp = abs(n);
        while(tmp>0){
            if((1&tmp) == 1) result = result*base;
            base = base*base;
            tmp = tmp >>1 ;
        }
        return n>0?result:1/result;
  }
```


