# HashMap线程安全问题

> hashmap是线程不安全的

但是线程不安全体现在哪里？

`get`方法肯定不会带来线程安全问题，问题出在`put`方法中，如果两个线程要`put`的key是不同的，其实也没有线程不安全问题，如果要`put`的key是相同的，那更没有问题了，因为`hashmap`是保证key唯一的，后插入的数据直接覆盖前面插入的，不会出现两个一样可以的数据。

**线程不安全就出现**`resize`中

*当两个线程操作一个hashmap，且两个线程都出应该resize了，其中一个线程已经完成了resize，并且把table更新为新的newTable，但是另一个线程还处于**`transfer`**方法中，最后会导致循环引用的问题。*

案例：

1、两个线程都要插入数据

![img](https://img-blog.csdnimg.cn/20200807150830859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5MDUxNDEz,size_16,color_FFFFFF,t_70)

2、插入后变成如下：

![img](https://img-blog.csdnimg.cn/20200807150916179.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5MDUxNDEz,size_16,color_FFFFFF,t_70)

3、接着两个线程又准备插入数据，而且两个线程要插入的位置都是在table[2]，此时出发resize条件，两个线程都进入了resize操作：

![img](https://img-blog.csdnimg.cn/20200807151015727.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5MDUxNDEz,size_16,color_FFFFFF,t_70)

4、线程A已经完成了resize并且更新了table为最新，线程B还刚开始transfer：

![img](https://img-blog.csdnimg.cn/20200807151201795.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5MDUxNDEz,size_16,color_FFFFFF,t_70)

5、线程B在transfer过程中，就出现了以下情况：

![img](https://img-blog.csdnimg.cn/20200807151314182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5MDUxNDEz,size_16,color_FFFFFF,t_70)

等线程B更新了table后，table就变为以下情况：

![img](https://img-blog.csdnimg.cn/20200807151353900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5MDUxNDEz,size_16,color_FFFFFF,t_70)

后面要执行get的时候，就进入死循环了。

JDK7下的Hashmap线程不安全的原理。

视频讲解：