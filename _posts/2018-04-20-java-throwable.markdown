---
layout: post
title: Java Throwable
date: 2018-04-20 00:20:20 +0300
description:
img:
tags: [Java]
---

Throwable 分为Error、Exception，Exception又分为不受检查的异常和受检查的异常

#### Error

Error 合理的应用程序不应该尝试获取的严重错误。一般是系统错误或底层资源的错误，不受检查，要捕获应该在系统级别

AnnotationFormatError 注解解释器尝试从一个class文件读取注解，当格式错误会抛此Error

AssertionError 这个是断言时的错误

LinkageError 一个类依赖另一个类时，后者在编译前一个类后发生了不兼容变化

VirtualMachineError 虚拟机Error，如内存溢出等等

#### Exception

Exception 合理的应用程序获取的异常信息。一般由编码者导致，不受检查和受检查2种，要捕获在应用级别

不受检查 Java编译器在编译的时候不会去检查，如RuntimeException和Error。 在手动抛出的时候，不用try...catch或throws

受检查 ava编译器在编译的时候会去检查,如IOException等等。在手动抛出的时候，必须用try...catch或throws其中一种

#### throw 和 throws

throw 用来抛出异常，如 throw new AssertionError();

throws 是用来标明当前函数可能抛出的异常

#### try-catch-finally

执行顺序
执行try块内代码，如果出现异常,进入catch代码块(前提是出现的异常必须是catch的异常的子类或本身，多个catch时按顺序捕获)。最终都会进入finally

注意
1. 如果finally 有return语句，会覆盖前面的执行结果；如果抛出异常，会覆盖前面的执行结果
2. 如果 try-catch 返回结果，finally操作了结果(并没有return，只是计算)，返回的结果还是try-catch 返回结果，不受改变

#### 其他

在Java编码中一般说的异常都指Exception，一般Error不在我们的处理中。
现在来对比断言和Exception的区别

在Java中已assert关键字来标识(默认是不开启的)
assert 条件:错误信息(可缺省) 当条件为false的时候，会抛出AssertionError

测试中 断言可以快速定位错误
发布运行中 保证程序的正确性和一致性(断言的条件除非常非常特殊的情况会出现false，这也是用Error的原因。如果false经常出现，那就应该用Exception了。断言的场景，就是很确定的感觉)

Exception 保证的是应用的健壮性