# 堆的分代GC过程

**新生代** 

新生代又细分为三个区：Eden区、SurvivorFrom、ServivorTo区，三个区的默认比例为：8：1：1。 大多数情况下，对象在新生代Eden区中分配。当Eden区没有足够的空间进行分配时，虚拟机将发起一次MinorGC。在新生代中，保存着大量的刚刚创建的对象，但是大部分的对象都是朝生夕死，所以在新生代中会频繁的进行MinorGC，进行垃圾回收。

Eden区：Java新创建的对象绝大部分会分配在Eden区（如果对象太大，则直接分配到老年代）。当Eden区内存不够的时候，就会触发MinorGC（新生代采用的是复制算法），对新生代进行一次垃圾回收。 

SurvivorFrom区和To区：在GC开始的时候，对象只会存在于Eden区和名为From的Survivor区，To区是空的，一次MinorGc过后，Eden区和SurvivorFrom区存活的对象会移动到SurvivorTo区中，然后会清空Eden区和SurvivorFrom区，并对存活的对象的年龄+1，如果对象的年龄达到15，则直接分配到老年代。MinorGC完成后，SurvivorFrom区和SurvivorTo区的功能进行互换。下一次MinorGC时，会把SurvivorTo区和Eden区存活的对象放入SurvivorFrom区中，并计算对象存活的年龄。 老年代 老年代主要存放应用中生命周期长的内存对象。

老年代比较稳定，不会频繁的进行MajorGC。而在MaiorGC之前才会先进行一次MinorGc，使得新生的对象进入老年代而导致空间不够才会触发。当无法找到足够大的连续空间分配给新创建的较大对象也会提前触发一次MajorGC进行垃圾回收腾出空间。 在老年代中，MajorGC采用了标记—清除算法：首先扫描一次所有老年代里的对象，标记出存活的对象，然后回收没有标记的对象。MajorGC的耗时比较长。因为要扫描再回收。MajorGC会产生内存碎片，当老年代也没有内存分配给新来的对象的时候，就会抛出OOM异常。

**永久代**

 永久代指的是永久保存区域。主要存放Class和Meta（元数据）的信息。Classic在被加载的时候被放入永久区域，它和存放的实例的区域不同，在Java8中，已经被移除，取而代之的是一个称之为“元数据区”（元空间）的区域。元空间和永久代类似，都是对JVM中规范中方法的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存的限制。类的元数据放入native memory，字符串池和类的静态变量放入java堆中。这样可以加载多少类的元数据就不再由MaxPermSize控制，而由系统的实际可用空间来控制。 采用元空间而不使用永久代的原因：

为了解决永久代的OOM问题，元数据和class对象存放在永久代中，容易出现性能问题和内存溢出。 类及方法的信息等比较难确定其大小，因此对于永久代大小指定比较困难，大小容易出现永久代溢出，太大容易导致老年代溢出（堆内存不变，此消彼长） 永久代会为GC带来不必要的复杂度，并且回收效率偏低。 MinorGC和Full GC 新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为Java对象大多都具有朝生夕灭的特性，所以MinorGC非常频繁，一般回收速度也比较快。 老年代GC（Major GC/Full GC）：指发生在老年代的GC，出现了Major GC，经常会伴随至少一次的Minor GC（但是这并不是绝对的）。Major GC的速度一般会比Minor GC慢10倍以上。