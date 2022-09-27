# CAS、volatile、乐观锁、悲观锁

## 1、CAS算法介绍

CAS（Compare and swap）比较与交换， 是一种有名的**无锁算法**，CAS的3个操作数内存值V旧的预期值A要修改的新值B**当且仅当内存值V和预期值A相同时，就将内存值V修改为新值B，否则不做任何操作**

当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值(**A和内存值V相同时，将内存值V修改为B)**，而其它线程都失败，失败的线程**并不会被挂起**，而是被告知这次竞争中失败，并可以再次尝试(否则什么都不做)缺点：CAS操作是先取出内存值，然后才将内存值与期望值比较的，这样可能会出现这样一个问题，假如线程1获取了内存值V，然后线程2也从内存中取出V，并且线程2进行了一些修改操作，将内存值修改为B，然后two又将内存值修改为V，当前线程的CAS操作无法分辨内存值V是否发生过变化，所以尽管CAS成功，但可能存在潜在的问题。解决使用AtomicStampedReference和AtomicMarkableReference实现标记的功能。小结：先**比较**是否相等，如果相等则**替换**(CAS算法)

CAS是项**乐观锁**技术，当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。

![img](https://pics5.baidu.com/feed/902397dda144ad34c3df3d0b7026a8f230ad8561.jpeg?token=13849a1ecf24a119fbf2ad3236a70506)

## 2、volatile介绍

volatile变量是一个更轻量级的同步机制，因为在使用这些变量时不会发生上下文切换和线程调度等操作，但是volatile变量也存在一些局限，比如不能用于构建原子的复合操作，因此当一个变量依赖旧值时就不能使用volatile变量，volatile内部已经做了synchronized。

**volatile只能保证变量对各个线程的可见性，但不能保证原子性**

保证**该变量对所有线程的可见性**在多线程的环境下：当这个变量修改时，**所有的线程都会知道该变量被修改了**，也就是所谓的“可见性”不保证原子性修改变量(赋值)**实质上**是在JVM中**分了好几步**，而**在这几步内(从装载变量到修改)，它是不安全的**。

## 3、乐观锁与悲观锁

悲观锁的代表是`synchronized`和`Lock`锁

1. 其核心思想是：线程只有占有了锁，才能去操作共享变量，每次只有一个线程占锁成功，获取锁失败的线程，都得停下来等待
2. 线程从运行到阻塞、再从阻塞到唤醒，设计线程上下文切换，如果频繁切换，影响性能
3. 实际上，线程在获取`synchronized`和`Lock`锁时，如果锁已被占用，都会做几次重试操作，减少阻塞的机会

乐观锁的代表是`AtomicInteger`，使用cas来保证原子性

1. 其核心思想是：无需加锁，每次只有一个线程能成功修改共享变量，其他失败的线程都不需要停止，不断重试直到成功
2. 由于线程一直运行，不需要阻塞，因此不涉及线程上下文切换
3. 它需要多核cpu支持，且线程数不应超过cpu核数

### 3.1. 乐观锁代码案例

```java
package com.mihu.demo.lock;

import sun.misc.Unsafe;

/**
 * 乐观锁案例
 */
public class SyncVsCas {
    static final Unsafe U = Unsafe.getUnsafe();
    static final long BALANCE = U.objectFieldOffset(Account.class, "balance");

    static class Account {
        volatile int balance = 10;
    }

    public static void sync(Account account) {

    }

    public static void cas(Account account) {

    }

    public static void main(String[] args) {
        Account account = new Account();
        while (true) {
            int o = account.balance;
            int n = o + 5;
            if (U.compareAndSwapInt(account, BALANCE, o, n)) {
                break;
            }
        }
    }

}
```

### 3.2. 乐观锁VS悲观锁代码

```java
package com.mihu.demo.lock;

import sun.misc.Unsafe;

/**
 * 乐观锁案例
 */
public class SyncVsCas {
    static final Unsafe U = Unsafe.getUnsafe();
    static final long BALANCE = U.objectFieldOffset(Account.class, "balance");

    static class Account {
        volatile int balance = 10;
    }

    public static void sync(Account account) {
        new Thread(() -> {
            synchronized (account) {
                int old = account.balance;
                int n = old - 5;
                account.balance = n;
            }
        }, "t1").start();

        new Thread(() -> {
            synchronized (account) {
                int o = account.balance;
                int n = o + 5;
                account.balance = n;
            }
        }, "t2").start();
    }

    public static void cas(Account account) {
        new Thread(() -> {
            while (true) {
                int o = account.balance;
                int n = o - 5;
                if (U.compareAndSwapInt(account, BALANCE, o , n)) {
                    break;
                }
            }
        }, "t1").start();

        new Thread(() -> {
            while (true) {
                int o = account.balance;
                int n = o - 5;
                if (U.compareAndSwapInt(account, BALANCE, o , n)) {
                    break;
                }
            }
        }, "t2").start();
    }

    public static void main(String[] args) {
        Account account = new Account();
        sync(account);
    }

}
```



## 4、Java中的原子操作

原子操作指的是在一步之内就完成而且不能被中断，原子操作在多线程环境中是线程安全的，无需考虑同步的问题。