# HashMap1.7与1.8

## 概述

HashMap是基于Map接口实现，元素以键值对的放的方式存储，并且允许使用null键和null值，因为key不允许重复，因此只能有一个键为null，另外HashMap不能保证放入元素的顺序，它是无序的，和放入的顺序并不能相同。

HashMap是线程不安全的。

底层由数组+链表实现，当链表太大的时候（阈值为8）在jdk1.8会将链表变成红黑树

## 继承关系

```Java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
```

## 基本属性

```Java
static final int DEAFULT_INITIAL_CAPACITY = 1 << 4; // 默认初始化大小16
static final float DEAFULT_LOAD_FACTORY = 0.75f; // 负载因子0.75，即数组达到 大小*0.75时进行扩容，扩容为2倍
static final Entry<?,?>[] EMPTY_TABLE = {}; // 初始化的默认数组
transient int size; // HashMap中元素的数量
int threshold; // 判断是否需要调整HashMap的容量
```

## HashMap和HashTable的区别

### 线程安全

1、HashTable是线程安全的，HashMap非线程安全

2、HashTable的实现方法里面都添加了synchronized关键字来确保线程安全，因此相对而言HashMap性能会高一些

3、我们平时使用时若无特殊需求建议使用 HashMap ，在多线程环境下若使用HashMap需要使用 Collection.synchronizedMap() 方法来获取一个线程安全的集合。

### 针对Null的不同

1、HashMap可以使用null作为key，而HashTable则不允许null作为key

2、虽说HashMap支持null值作为key，不过建议还是尽量避免这样使用，因为一旦不小心使用了，若因此引发一些问题，排查起来很是费事

### 继承结构

1、HashMap是对Map接口的实现

2、HashTable实现了Map接口和Dictionary抽象类

### 初识容量与扩容

1、HashMap的初始容量为16，HashTable初始容量为11，两者的填充因子默认都是0.75

2、HashMap扩容时是当前容量翻倍即：`capacity * 2`，HashTable扩容时容量翻倍+1即：`capacity * 2 + 1`

### 两者计算hash的方法不同

HashTable计算Hash是直接使用key的hashcode对table数组的长度直接进行取模

```Java
int hash = key.hashCode();
int index = (hash & 0x7FFFFFFF) % tab.length;
```

HashMap计算hash对key的hashcode进行了二次hash，以获得更好的散列值，然后对table数组长度取模

```Java
int hash = hash(key.hashCode());
int i = indexFor(hash, table.length);
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
static int indexFor(int h, int length) {
    return h & (length - 1);
}
```

## HashMap的数据存储结构

### HashMap由数组和链表来实现对数据的存储

HashMap采用Entry数组来存储key-value对，每一个键值对组成了一个Entry实体，Entry类实际上是一个单向的链表结构，它具有Next指针，可以连接下一个Entry实体，以此来解决Hash冲突的问题。

数组存储区间是连续的，占用内存严重，故空间复杂的很大。但数组的二分查找时间复杂度小，为O(1)；数组的特点是：寻址容易，插入和删除困难；

链表存储区间离散，占用内存比较松散，故空间复杂度很小，但时间复杂度很大，达O(N)；链表的特点是：寻址困难，插入和删除容易；

数据结构由数组+链表组成，一个长度为16的数组中，每个元素存储的是一个链表的头结点。那么这些元素是按照什么样的规则存储到数组中呢，一般情况是通过 hash(key.hashCode))%len 获得，也就是元素的key的哈希值对数组长度取模得到，比如。12%16=12, 28%16=12, 108%16=12, 140%16=12。所以12、28、108以及140都存储在数组下标为12的位置。

HashMap里面实现一个静态内部类 Entry，其重要的属性有hash， key， value， next

HashMap里面用到链式数据结构的一个概念，Entry类里面有一个next属性，作用是指向下一个Entry，打个比方，第一个键值对A进来，通过计算其key的hash得到的index=0，记做：Entry[0] = A。一会又进来一个键值对B，通过计算其index也等于0，HashMap会这样做：B.next = A, Entry[0] = B，如果又进来C，index也等于0，那么C.next = B, Entry[0] = C；这样我们发现index = 0的地方其实存储了A、B、C三个键值对，他们通过next这个属性链接在一起。也就是说数组中存储的是最后插入的元素。

```Java
public V put(K key, V value) {
    if (key == null)
        return putForNullKey(value); //null总是放在数组的第一个链表中
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
    //遍历链表
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        //如果key在链表中已存在，则替换为新value
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}

void addEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e); //参数e, 是Entry.next
    //如果size超过threshold，则扩充table大小。再散列
    if (size++ >= threshold)
        resize(2 * table.length);
}
```

## 重要方法深度解析

### 构造方法

```Java
HashMap() // 无参构造
HashMap(int initialCapacity)  //指定初始容量的构造方法 
HashMap(int initialCapacity, float loadFactor) //指定初始容量和负载因子
HashMap(Map<? extends K,? extends V> m)  //指定集合，转化为HashMap 
```

HashMap提供了四个构造方法，构造方法中，依靠第三个方法来执行的，但是前三个方法都没有进行数组的初始化操作，即使调用了构造方法此时存放HashMap中数组元素的table表长度依旧为0，在第四个构造方法中调用了inflateTable()方法完成了table的初始化操作，并将m中的元素添加到HashMap中。

### 添加方法`put()`

```Java
public V put(K key, V value) {
        if (table == EMPTY_TABLE) { //是否初始化
            inflateTable(threshold);
        }
        if (key == null) //放置在0号位置
            return putForNullKey(value);
        int hash = hash(key); //计算hash值
        int i = indexFor(hash, table.length);  //计算在Entry[]中的存储位置
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i); //添加到Map中
        return null;
}
```

在该方法中，添加键值对时，首先进行table是否初始化的判断，如果没有进行初始化（分配空间，Entry[] 数组的长度）。然后进行key是否为null的判断，如果key == null，放置在 Entry[] 的0号位置，计算在 Entry[] 数组的存储位置，判断该位置上是否已有元素，如果已有元素存在，则遍历该Entry[] 数组位置上的单向链表，判断 key 是否存在，如果 key 已经存在，则由新的value值，替换节点旧的 value 值，并将旧的 value 值返回，如果 key 不存在与 HashMap 中，程序继续向下执行。将 key-value ，生成 Entry实体，添加到 HashMap 的 Entry[] 数组中。

### addEntry()

```Java
/*
 * hash hash值
 * key 键值
 * value value值
 * bucketIndex Entry[]数组中的存储索引
 **/ 
void addEntry(int hash, K key, V value, int bucketIndex) {
     if ((size >= threshold) && (null != table[bucketIndex])) {
         resize(2 * table.length); //扩容操作，将数据元素重新计算位置后放入newTable中，链表的顺序与之前的顺序相反
         hash = (null != key) ? hash(key) : 0;
         bucketIndex = indexFor(hash, table.length);
     }

    createEntry(hash, key, value, bucketIndex);
}
void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```

添加到方法的具体操作，在添加之前先进行容量的判断，如果当前容量达到了阈值，并且需要存储到 Entry[] 数组中，先进行扩容操作，扩容的容量为table长度的2倍，重新计算hash值，和数组存储的位置，扩容后的链表顺序和扩容前的链表顺序相反，然后将新添加的Entry实体存放到当前Entry[]位置链表的头部。在1.8之前，新插入的元素都是放在了链表的头部位置，但是这种操作在高并发的环境下容易导致死锁，所以在1.8之后，新插入的元素都放在了链表的尾部。

### 获取方法：get

```Java
public V get(Object key) {
     if (key == null)
         //返回table[0] 的value值
         return getForNullKey();
     Entry<K,V> entry = getEntry(key);

     return null == entry ? null : entry.getValue();
}
final Entry<K,V> getEntry(Object key) {
     if (size == 0) {
         return null;
     }

     int hash = (key == null) ? 0 : hash(key);
     for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
         Object k;
         if (e.hash == hash &&
             ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
      }
     return null;
}
```

在get方法中，首先计算hash值，然后调用indexFor()方法得到该key在table中的存储位置，得到该位置的单向链表，遍历链表找到key和指定key内容相等的 Entry，返回 entry.value 值。

### 删除方法

```Java
public V remove(Object key) {
     Entry<K,V> e = removeEntryForKey(key);
     return (e == null ? null : e.value);
}
final Entry<K,V> removeEntryForKey(Object key) {
     if (size == 0) {
         return null;
     }
     int hash = (key == null) ? 0 : hash(key);
     int i = indexFor(hash, table.length);
     Entry<K,V> prev = table[i];
     Entry<K,V> e = prev;

     while (e != null) {
         Entry<K,V> next = e.next;
         Object k;
         if (e.hash == hash &&
             ((k = e.key) == key || (key != null && key.equals(k)))) {
             modCount++;
             size--;
             if (prev == e)
                 table[i] = next;
             else
                 prev.next = next;
             e.recordRemoval(this);
             return e;
         }
         prev = e;
         e = next;
    }

    return e;
}
```

删除操作，先计算指定key的hash值，然后计算出table中的存储位置，判断当前位置是否 Entry 实体存在，如果没有直接返回，若当前位置有 Entry 实体存在，则开始遍历链表，定义了三个 Entry 引用，分别为 pre、e、next。在循环遍历的过程中，首先判断pre和e是否相等，若相等，表明table的当前位置只有一个元素，直接将table[i] = next = null。若形成了 pre -> e -> next 的连接关系，判断 e 的key是否和指定的key相等，若相等则让 pre -> next，e失去引用。

### containsKey

```Java
public boolean containsKey(Object key) {
        return getEntry(key) != null;
}
final Entry<K,V> getEntry(Object key) {
    int hash = (key == null) ? 0 : hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

containsKey方法是先计算hash然后使用hash和table.length取模得到index值，遍历table[index]元素查找是否包含key相同的值。

### ContainsKey

```Java
public boolean containsValue(Object value) {
    if (value == null)
            return containsNullValue();

    Entry[] tab = table;
        for (int i = 0; i < tab.length ; i++)
            for (Entry e = tab[i] ; e != null ; e = e.next)
                if (value.equals(e.value))
                    return true;
    return false;
}
```

containsValue方法就比较粗暴了，就是直接遍历所有元素直到找到value，由此可见HashMap的containsValue方法本质上和普通数组和list的contains方法没什么区别，别指望它会向containsKey那么高效。

## JDK 1.8的改变

### HashMap采用数组+链表+红黑树实现

在jdk1.8中HashMap的实现方式做了一些改变，但是基本思想还是没有一样，只是在一些地方做了优化。

数据结构的存储由数组+链表的方式，变化为数组+链表+红黑树的存储方式，当链表长度超过阈值（8）时，将链表转换为红黑树。

### put方法

```Java
public V put(K key, V value) {
    //调用putVal()方法完成
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //判断table是否初始化，否则初始化操作
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //计算存储的索引位置，如果没有元素，直接赋值
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        //节点若已经存在，执行赋值操作
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //判断链表是否是红黑树
        else if (p instanceof TreeNode)
            //红黑树对象操作
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //为链表，
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //链表长度8，将链表转化为红黑树存储
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //key存在，直接覆盖
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    //记录修改次数
    ++modCount;
    //判断是否需要扩容
    if (++size > threshold)
        resize();
    //空操作
    afterNodeInsertion(evict);
    return null;
}
```

## 为什么要用红黑树

链表太大的时候，需要遍历的时间长，影响hashMap的性能，使用红黑树遍历效率高

## 什么时候转为红黑树

1. 链表长度超过8
2. 整个数组的长度要 ≥ 64

两个条件都满足才会转为红黑树

## 为什么不是一开始就使用红黑树

链表短的时候，性能是高于红黑树的

变成树后，占用内存也比较高，链表的数据结构是Node，红黑树数据结构是TreeNode，TreeNode的成员变量比Node的成员变量多，所以占用内存也多。

红黑树用来避免DoS攻击，防止链表超长时性能下降，树化是偶然情况

hash表的查找，更新的时间复杂度是O(1)，而红黑树的查找，更新的时间复杂度是O(log2n)，TreeNode占用空间也比普通Node的大，如非必要，尽量还是使用链表。

## 树化阈值为什么是8

hash值如果足够随机，则在hash表内按泊松分布，在负载因子0.75的情况下，长度超过8的链表出现概率是0.00000006，选择8就是为了让树化几率足够小。

## 何时会退化为链表

退化情况1：在扩容时如果拆分树时，树元素个数≤6则会退化链表

退化情况2：remove树节点时，若root、root.left、root.right、root.left.left有一个为null，也会退化为链表

## 索引如何计算？hashCode都有了，为何还要提供hash()方法？数组容量为何是2的n次幂？

1. 计算对象的hashCode()，再进行调用HashMap的hash()方法进行二次哈希，最后`&(capacity - 1)`得到索引
2. 二次hash()是为了综合高位数据，让哈希分布更为均匀
3. 计算索引时，如果是2的n次幂可以使用位于运算代替取模，效率更高；扩容时`hash & oldCap == 0`的元素留在原来位置，否则`新位置 = 旧位置 + oldCap`
4. 但上述3点都是为了配合容量为2的n次幂时的优化手段，例如HashTable的容量就不是2的n次幂，并不能说哪种设计更优，应该是设计者综合了各种因素，最终选择了使用2的n次幂作为容量。

## `put()`方法JDK7和JDK8有何不同

`put()`流程：

1. HashMap是懒惰创建数组的，首次使用才创建数组
2. 计算索引（桶下标）
3. 如果桶下标还没人占用，创建Node占位返回
4. 如果桶下标已经有人占用
   1. 已经是TreeNode走红黑树的添加或更新逻辑
   2. 是普通Node，走链表的添加或更新逻辑，如果链表长度超过树化阈值，走树化逻辑
5. 返回前检查容量是否超过阈值，一旦超过进行扩容

不同：

1. 链表插入节点时，1.7是头插法，1.8是尾插法
2. 1.7是大于等于阈值且没有空位时才扩容（插入位置的链表没有元素，且大于等于阈值扩容），而1.8是大于阈值就扩容
3. 1.8在扩容计算Node索引时，会优化

## 加载因子为何默认是0.75f

1. 在空间占用与查询时间之间取得较好的权衡
2. 大于这个值，空间节省了，但链表就会比较长影响性能
3. 小于这个值，冲突减少了，但扩容就会更频繁，空间占用多

## 多线程下会出现啥问题

1. 扩容死链(1.7)
2. 数据错乱(1.7，1.8)：在链表初始化的时候，如果多线程会造成双方都判断需要初始化链表，则有一个链表的数据会被覆盖

## Key能否为null，作为key的对象有什么要求

1. HashMap的key可以为null，但Map的其他实现则不然
2. 作为key的对象，必须实现hashCode和equals，并且key的内容不能修改（不可变）

## String对象的`hashCode()`如何设计的，为啥每次乘的是31

目标是达到较为均匀的散列效果，每个字符串的hashCode足够独特

1. 字符串中的每个字符都可以表现为一个数字，称为Si，其中i的范围是0~n-1
2. 散列公式为：`S0*31 n-1次方+S1*31 n-2次方+...Si*31 n-1-i次方+...Sn-1*31 0次方 `
3. 31代入公式有较好的散列特性，并且31*h可以被优化为
   1. 即32*h-h
   2. 即2的5次方*h-h
   3. 即h << 5 - h