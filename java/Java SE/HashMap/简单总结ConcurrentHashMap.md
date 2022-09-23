# 简单总结ConcurrentHashMap

## ConcurrentHashMap的优势

在并发使用到HashMap的时候，往往不建议直接用HashMap，因为HashMap在并发写数据的时候容易因为rehash的过程产生环形链表的情况。所以在并发使用Map结构时，一般建议使用ConcurrentHashMap。

## JDK1.7的ConcurrentHashMap

在JDK1.7中ConcurrentHashMap采用了**数组+Segment+分段锁**的方式实现。

- **Segment(分段锁)：** ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表,同时又是一个ReentrantLock（Segment继承了ReentrantLock）。
- **内部结构：** ConcurrentHashMap使用分段锁技术，将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问，能够实现真正的并发访问。如下图是ConcurrentHashMap的内部结构图：

![img](https://upload-images.jianshu.io/upload_images/20782304-35290e5426e7faab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/809/format/webp)

> ConcurrentHashMap定位一个元素的过程需要进行两次Hash操作。第一次Hash定位到Segment，第二次Hash定位到元素所在的链表的头部。

## JDK1.8之后的ConcurrentHashMap

参考1.8的HashMap的实现，采用了**数组+链表+红黑树**的实现方式来设计，内部大量采用CAS操作。

并发控制使用**`synchronized`和CAS来操作 **，整个看起来就像是优化过且线程安全的hashMap，虽然在1.8中还能看到Segment的数据结构，但只是为了兼容旧版本。

1.8的Node节点中value和next都用volatile修饰，保证并发的可见性。

可以理解为，synchronized只锁定当前链表或红黑树的首节点，这样只要hash不冲突，就不会产生并发。

![img](https://upload-images.jianshu.io/upload_images/20782304-f7ba6ac0add69cb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/867/format/webp)

## ConcurrentHashMap和HashTable的区别

Hashtable和1.8之前的hashmap的底层数据结构类似，都是采用**数组+链表**的形式，数组是hashmap的主体，链表则是主要为了解决hash冲突而存在的。

hashtable（同一把锁）：使用synchronized来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用put添加元素，另一个线程不能使用put添加元素，也不能使用get，竞争会越来越激烈效率越低。

> ConcurrentHashMap不论时1.7还是1.8，他的效率都比hashtable高，主要还是因为hashtable使用了一种全表加锁的方式。

![img](https://upload-images.jianshu.io/upload_images/20782304-8866c91f63937053.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/705/format/webp)