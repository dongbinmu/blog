---
layout: post
title: 同步器接口及实现demo
date: 2018-04-21 00:20:20 +0300
description:
img:
tags: [Java,Lock]
---

同步器设计是基于模版方法的，可以继承同步器来实现自己的锁。同步器主要关键是设置state的值，来区分是否能获取资源


同步器重写方法

|方法|描述|
|-|-|
|protected boolean tryAcquire(int arg) |独占式获取同步状态，实现该方法要查询当前状态并判断同步状态是否符合预期，然后在通过CAS设置同步状态|
|protected boolean tryRelease(int arg)|独占式释放同步状态，等待获取同步状态的线程将有机会获取同步状态|
| protected boolean isHeldExclusively()|当前同步器是否在独占模式下被线程占用，一般改方法表示是否被当前线程所独占|
|protected int tryAcquireShared(int arg)|共享式获取同步状态，返回值大于等于0，表示获取成功，反之失败|
|protected boolean tryReleaseShared(int arg)|共享式释放同步状态|

同步器提供的模版方法

|方法|描述|
|-|-|
|void acquire(int arg)|独占式获取同步状态，如果当前线程获取同步状态成功，则返回，否则将会进入同步队列等待，这个方法会调用重写的tryAcquire(arg)|
|void acquireInterruptibly(int arg)|与acquire(arg)相同，但是改方法响应中断，当前线程未获取到同步状态而进入同步队列中，如果当前线程被中断，会抛出InterruptedException异常|
|boolean tryAcquireNanos(int arg, long nanosTimeout)|在acquireInterruptibly上增加了超时限制，如果当前线程在超时事件内没有获取到线程 返回false,获取到返回true|
|boolean release(int arg)|独占式释放同步状态，释放后会将同步队列的第一个节点包含的线程唤醒|
| void acquireShared(int arg)|共享式获取同步状态，如果未获取到则进入同步队列，与独占式的区别是可以有多个线程取到同步状态|
|void acquireSharedInterruptibly(int arg)|该方法响应中断|
|boolean tryAcquireSharedNanos|增加了超时限制|
|boolean releaseShared(int arg)|共享式释放同步状态|

#### demo

互斥锁实现Mutex

```java
/**
 * Created by dongbin on 2018/4/20.
 */
public class Mutex implements Lock {

    //自定义同步器
    private static class Sync extends AbstractQueuedSynchronizer {

        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        //尝试获取锁
        @Override
        protected boolean tryAcquire(int arg) {
            Assert.check(arg==1);
            if (compareAndSetState(0, arg)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int arg) {
            Assert.check(arg==1);
            if (getState() == 0) throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }


        Condition newCondition() {
            return new ConditionObject();
        }
    }

    private final Sync sync = new Sync();

    @Override
    public void lock() {
        sync.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1,unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.release(1);
    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

共享锁(2个线程进入)

```java
/**
 * Created by dongbin on 2018/4/20.
 */
public class TwinsLock implements Lock {

    private static final class Sync extends AbstractQueuedSynchronizer {

        Sync(int count) {
            if (count != 2) {
                throw new IllegalMonitorStateException("state 只能为2");
            }
            setState(count);
        }

        @Override
        protected boolean tryReleaseShared(int arg) {
            Assert.check(arg == 1);
            for (; ; ) {
                int current = getState();
                int newCount = current + arg;
                Assert.check(newCount <= 2);
                if (compareAndSetState(current, newCount)) {
                    return true;
                }
            }
        }

        @Override
        protected int tryAcquireShared(int arg) {
            Assert.check(arg == 1);
            for (; ; ) {
                int current = getState();
                int newCount = current - arg;
                if (newCount < 0 || compareAndSetState(current, newCount)) {
                    return newCount;
                }
            }
        }

        final Condition newConditionObject() {
            return new ConditionObject();
        }
    }

    private Sync sync = new Sync(2);

    @Override
    public void lock() {
        sync.acquireShared(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquireShared(1) >= 0;
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.release(1);
    }

    @Override
    public Condition newCondition() {
        return sync.newConditionObject();
    }
}

```