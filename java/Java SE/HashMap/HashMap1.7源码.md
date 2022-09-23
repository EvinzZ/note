# HashMap1.7源码

## 继承关系

```Java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
     private static final long serialVersionUID = 362498820763181265L;
    }
```

HashMap不仅继承了AbstractMap，而且实现了Map、Cloneable和Serializable接口，所以HashMap也可以序列化。 注：HashMap不是同步的，是非同步的，因此HashMap是不安全的。但是可以通过Collections类的静态方法synchronizedMap获得线程安全的HashMap。

```Java
Map pool = Collections.synchronizedMap(new HashMap());
```

## 存储结构

HashMap1.7的底层主要是基于数组和链表来实现的。HashMap的查询速度很快，这主要是源于HashMap的存储方式是通过计算散列表来实现的。通过Key和Value来计算hash值，然后通过不同的hash值来选择不同的数组进行存储。这里就要提到hash冲突（对不同的关键字，可能得到同一个散列地址，即同一个数组下标，这种现象称为冲突）。HashMap的底层是通过链表来解决hash冲突的。 通过

![img](https://secure2.wostatic.cn/static/az7vJzpBbLkAAy6DY4arbt/image.png)

**注**：hash数组中实际上存储的是每一个链表的头结点。

HashMap存储的数组里面存放的是Entry<K,V>实体；

### **Entry实体**

Entry是一个单向链表，它实现了Map.Entry接口；

```Java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next; //指向下一个节点
    int hash;
 
    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }
 
    public final K getKey() {
        return key;
    }
 
    public final V getValue() {
        return value;
    }
 
    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
 
    public final boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;
        Object k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            Object v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }
 
    public final int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }
 
    public final String toString() {
        return getKey() + "=" + getValue();
    }
 
    void recordAccess(HashMap<K,V> m) {
    }
 
    void recordRemoval(HashMap<K,V> m) {
    }
}
```

这里可以把HashMap看做一个存储Entry的数组，Entry对象包含了键和值，其中next也是一个Entry对象，用来处理hash冲突的，形成一个链表。

### HashMap的成员属性

```Java
//默认初始容量是16，必须是2的幂
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
 
//最大容量（必须是2的幂且小于2的30次方，传入容量过大会被这个值替换）
static final int MAXIMUM_CAPACITY = 1 << 30;
 
//默认加载因子，所谓加载因子是指哈希表在其容量自动增加之前可以达到多满的一种尺度
static final float DEFAULT_LOAD_FACTOR = 0.75f;
 
//存储Entry的默认空数组
static final Entry<?,?>[] EMPTY_TABLE = {};
 
//存储Entry的数组，长度为2的幂。HashMap采用拉链法实现的，每个Entry的本质是个单向链表
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
 
//HashMap的大小，即HashMap存储的键值对数量
transient int size;
 
//HashMap的阈值，用于判断是否需要调整HashMap的容量
int threshold;
 
//加载因子实际大小
final float loadFactor;
 
//HashMap被修改的次数，用于fail-fast机制
transient int modCount;
```

### **默认加载因子DEFAULT_LOAD_FACTOR为什么是0.75？**

若加载因子设置过大，空间利用率越高，但是冲突机会增加。链表就会变得很长，那么查找速率就会下降。

若加载因子设置过小，空间利用率下降，链表中数据变得稀疏。查找速率变高。

其实就是对空间利用率和查找速率做了一个平衡，找到0.75这个比例比较合适。

### **HashMap的构造方法**

```Java
public HashMap(int initialCapacity, float loadFactor) {//带有初始容量和加载因子
  //确保容量数字合法
  if (initialCapacity < 0) 
    throw new IllegalArgumentException("Illegal initial capacity: " +
                       initialCapacity);
  if (initialCapacity > MAXIMUM_CAPACITY)
    initialCapacity = MAXIMUM_CAPACITY;
  if (loadFactor <= 0 || Float.isNaN(loadFactor))
    throw new IllegalArgumentException("Illegal load factor: " +
                       loadFactor);
 
  this.loadFactor = loadFactor;
        //将阈值设置为初始容量，这里不是真正的阈值，是为了扩展table的，后面这个阈值会重新计算
        threshold = initialCapacity; 
  init();//一个空方法用于未来的子对象扩展</span></span>
}
 
public HashMap(int initialCapacity) { //带有初始容量，加载因子设为默认值
  this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
 
public HashMap() { //初始容量和加载因子均为默认值
  this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}
 
//构造一个映射关系与指定 Map 相同的新 HashMap
public HashMap(Map<? extends K, ? extends V> m) {
  this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
          DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
  inflateTable(threshold);
 
  putAllForCreate(m);
}
```

### **HashMap的put()方法**

```Java
public V put(K key, V value) {
  if (table == EMPTY_TABLE) { //如果哈希表没有初始化(table为空)
    inflateTable(threshold); //用构造时的阈值(其实就是初始容量)扩展table
  }
  //如果key==null，就将value加到table[0]的位置
  //该位置永远只有一个value，新传进来的value会覆盖旧的value
  if (key == null) 
    return putForNullKey(value);
 
  int hash = hash(key); //根据键值计算hash值
 
  int i = indexFor(hash, table.length); //搜索指定hash在table中的索引
 
  //循环遍历Entry数组，若该key对应的键值对已经存在，则用新的value取代旧的value
  for (Entry<K,V> e = table[i]; e != null; e = e.next) {
    Object k;
    if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
      V oldValue = e.value;
      e.value = value;
      e.recordAccess(this);
      return oldValue; //并返回旧的value
    }
  }
 
  modCount++;
  //如果在table[i]中没找到对应的key，那么就直接在该位置的链表中添加此Entry
  addEntry(hash, key, value, i);
  return null;
}
```

首先检测table是不是为空table，如果是空table，说明并没有给table初始化，所以调用inflateTable(threadshold)方法给table初始化。

### **inflateTable()方法**

```Java
//扩展table
private void inflateTable(int toSize) {
  // Find a power of 2 >= toSize
  int capacity = roundUpToPowerOf2(toSize); //获取和toSize最接近的2的幂作为容量
  //重新计算阈值 threshold = 容量 * 加载因子
  threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
  table = new Entry[capacity]; //用该容量初始化table
  initHashSeedAsNeeded(capacity);
}
//将初始容量转变成2的幂
private static int roundUpToPowerOf2(int number) {
    // assert number >= 0 : "number must be non-negative";
    return number >= MAXIMUM_CAPACITY
            ? MAXIMUM_CAPACITY //如果容量超过了最大值，设置为最大值
            //否则设置为最接近给定值的2的次幂数
            : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
}
```

在inflateTable方法内，首先初始化数组容量大小，数组容量永远是2的幂。所以调用roundUpToPowerOf2方法将传进来的容量转换成最接近2的次幂的值，然后重新计算阈值threadshold = 容量*加载因子，最后初始化table。所以刚开始初始化table不是在HashMap的构造函数里，因为构造函数中仅仅简单的将传进去的容量作为阈值。真正初始化table是在第一次往HashMap中put数据的时候。

初始化table后，要继续存储数据。table中存储的是Entry实体，而put方法中传递的key和value。首先，想要找到table数组中要存放的位置，然后将key和value封装到Entry中存入。

put方法中当key为null时，调用putForNullKey方法。

### **putForNullKey方法**

```Java
//传进key==null的Entry
private V putForNullKey(V value) {
  for (Entry<K,V> e = table[0]; e != null; e = e.next) {
    if (e.key == null) { 
      V oldValue = e.value;
      e.value = value;
      e.recordAccess(this);
      return oldValue;
    }
  }
  modCount++;
  //如果table[0]处没有key为null
  addEntry(0, null, value, 0);//如果键为null的话，则hash值为0
  return null;
}
```

首先根据nullhash值，定位到table[0]处，然后 依次查询key==null的键，如果有，则覆盖value值，并返回原来的value值。如果没有，则调用addEntry方法，将空键和值封装到Entry中存入到table[0]的位置。

### **addEntry方法**

```Java
//向HashMap中添加Entry
void addEntry(int hash, K key, V value, int bucketIndex) {
  if ((size >= threshold) && (null != table[bucketIndex])) {
    resize(2 * table.length); //扩容2倍
    hash = (null != key) ? hash(key) : 0;
    bucketIndex = indexFor(hash, table.length);
  }
 
  createEntry(hash, key, value, bucketIndex);
}
//创建一个Entry
void createEntry(int hash, K key, V value, int bucketIndex) {
  Entry<K,V> e = table[bucketIndex];//先把table中该位置原来的Entry保存
  //在table中该位置新建一个Entry，将原来的Entry挂到该Entry的next
  table[bucketIndex] = new Entry<>(hash, key, value, e);
  //所以table中的每个位置永远只保存一个最新加进来的Entry，其他Entry是一个挂一个，这样挂上去的
  size++;
}
```

从createEntry方法中可以看出，第一个参数是hash值，中间两个是key和value，最后一个是插入table的索引位置。插入之前先判断容量是否足够，若不够，HashMap中是2倍扩容。若够了，addEntry中先计算hash值，然后通过调用indexFor方法返回在索引的位置，这两个方法如下：

```Java
final int hash(Object k) {
  int h = hashSeed;
  if (0 != h && k instanceof String) {
    return sun.misc.Hashing.stringHash32((String) k);
  }
 
  h ^= k.hashCode();
        // 预处理hash值，避免较差的离散hash序列，导致table没有充分利用
  h ^= (h >>> 20) ^ (h >>> 12);
  return h ^ (h >>> 7) ^ (h >>> 4);
}

static int indexFor(int h, int length) {
  return h & (length-1);
}
```

> indexFor方法返回索引的位置，里面只做了一件事：h &(length-1)。这究竟做了什么？为什么这句能解释容量必须为2的幂呢？我们详细分析下：首先，h & (length-1)相当于h % length，但是h % length效率比较低（HashTable中是这儿干的）。为啥h& (length-1)相当于h %length呢？现在假设length为2的幂，那么length就可以表示成100…00的形式（表示至少1个0），那么length-1就是01111…11。对于任意小于length的数h来说，与01111…11做&后都是h本身，对于h=length来说，&后结果为0，对于大于length的数h，&过后相当于h-j*length，也就是h % length。这也就是为啥容量必须为2的幂了，为了优化，好做&运算，效率高。其次，length为2的次幂的话，是偶数，这样length-1为奇数，奇数的最后一位是1，这样便保证了h & (length-1)的最后一位可能为0也可能为1（取决于h的值），即结果可能为奇数，也可能为偶数，这样便可以保证散列的均匀性，即均匀分布在数组table中；而如果length为奇数的话，很明显length-1为偶数，它的最后一位是0，这样h &(length-1)的最后一位肯定为0，级只能为偶数，这样任何hash值都会被映射到数组的偶数下标位置上，这便浪费了近一半的空间！因此，length去2的整数次幂，也是为了使不同hash值发生碰撞的概率较小，这样就能使元素在哈希表中均匀的散列。

### **HashMap的get方法**

```Java
public V get(Object key) {
  if (key == null)
    return getForNullKey(); //hey==null时，从table[0]中取
  Entry<K,V> entry = getEntry(key);//key!=null->getEntry
 
  return null == entry ? null : entry.getValue();
}
 
private V getForNullKey() {
  if (size == 0) {
    return null;
  }
  for (Entry<K,V> e = table[0]; e != null; e = e.next) {
    if (e.key == null)
      return e.value;//从table[0]中取key==null的value值
  }
  return null;
}
 
final Entry<K,V> getEntry(Object key) {
  if (size == 0) {
    return null;
  }
  //取值与上面put中传值相反
  int hash = (key == null) ? 0 : hash(key);
  for (Entry<K,V> e = table[indexFor(hash, table.length)];
     e != null;
     e = e.next) {
    Object k;
    // 如果hash值相等，并且key相等则证明这个Entry里的东西是我们想要的
    if (e.hash == hash &&
      ((k = e.key) == key || (key != null && key.equals(k))))
      return e;
  }
  return null;
}
```

HashMap中的其他方法：

```Java
//返回当前HashMap的key-value映射数，即Entry数量
public int size() {
  return size;
}
 
//判断HashMap是否为空，size==0表示空
public boolean isEmpty() {
  return size == 0;
}
 
//判断HashMap中是否包含指定键的映射
public boolean containsKey(Object key) {
  return getEntry(key) != null; //getEntry方法在上面已经拿出来分析了
}
 
//看看需不需要创建新的Entry
private void putForCreate(K key, V value) {
  // 如果key为null，则定义hash为0，否则用hash函数预处理  
  int hash = null == key ? 0 : hash(key);
  int i = indexFor(hash, table.length);
  
  //遍历所有的Entry
  for (Entry<K,V> e = table[i]; e != null; e = e.next) {
    Object k;
    //如果有hash相同，且key相同，那么则不需要创建新的Entry，退出
    if (e.hash == hash &&
      ((k = e.key) == key || (key != null && key.equals(k)))) {
      e.value = value;
      return;
    }
  }
 
  createEntry(hash, key, value, i);//否则需要创建新的Entry
}
 
//根据已有的Map创建对应的Entry
private void putAllForCreate(Map<? extends K, ? extends V> m) {
  for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
    putForCreate(e.getKey(), e.getValue());
}
 
//用新的容量来给table扩容
void resize(int newCapacity) {
  Entry[] oldTable = table; //保存old table
  int oldCapacity = oldTable.length; //保存old capacity
  // 如果旧的容量已经是系统默认最大容量了，那么将阈值设置成整形的最大值，退出  
  if (oldCapacity == MAXIMUM_CAPACITY) {
    threshold = Integer.MAX_VALUE;
    return;
  }
 
  //根据新的容量新建一个table
  Entry[] newTable = new Entry[newCapacity];
  //将table转换成newTable
  transfer(newTable, initHashSeedAsNeeded(newCapacity));
  table = newTable;
  //设置阈值
  threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
 
//将所有的Entry移到新的table中
void transfer(Entry[] newTable, boolean rehash) {
  int newCapacity = newTable.length;
  for (Entry<K,V> e : table) {//获得原来table中的所有Entry
    while(null != e) {
      Entry<K,V> next = e.next;
      if (rehash) {
        e.hash = null == e.key ? 0 : hash(e.key);
      }
      int i = indexFor(e.hash, newCapacity);
      //下面两句跟createEntry方法中原理一样的
      e.next = newTable[i];//设置e.next为newTable[i]保存的Entry
      newTable[i] = e; //将e设置为newTable[i]
      e = next; //设置e为下一个Entry，继续上面while循环
    }
  }
}
 
//将指定的Map中所有映射复制到现有的HashMap中,这些映射关系将覆盖当前HashMap中针对指定键相同的映射关系
//有点拗口，其实就是跟put的原理一样，key如果存在，就替换现有key对应的value
public void putAll(Map<? extends K, ? extends V> m) {
  //统计下需要复制多少个映射关系
  int numKeysToBeAdded = m.size();
  if (numKeysToBeAdded == 0)
    return;
  //如果table还没初始化，先用刚刚统计的复制数去初始化table
  if (table == EMPTY_TABLE) {
    inflateTable((int) Math.max(numKeysToBeAdded * loadFactor, threshold));
  }
 
  //如果要复制的数目比阈值还要大
  if (numKeysToBeAdded > threshold) {
    //需要先resize
    int targetCapacity = (int)(numKeysToBeAdded / loadFactor + 1);
    if (targetCapacity > MAXIMUM_CAPACITY)
      targetCapacity = MAXIMUM_CAPACITY;
    int newCapacity = table.length;
    while (newCapacity < targetCapacity)
      newCapacity <<= 1;
    if (newCapacity > table.length)
      resize(newCapacity);
  }
  //开始复制
  for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
    put(e.getKey(), e.getValue());//其实就是一个个put进去
}
 
//根据指定的key删除Entry，返回对应的value
public V remove(Object key) {
  Entry<K,V> e = removeEntryForKey(key);
  return (e == null ? null : e.value);
}
//根据指定的key，删除Entry,并返回对应的value
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
      if (prev == e) //如果删除的是table中的第一项的引用
        table[i] = next;//直接将第一项中的next的引用存入table[i]中
      else
        prev.next = next; //否则将table[i]中当前Entry的前一个Entry中的next置为当前Entry的next
      e.recordRemoval(this);
      return e;
    }
    prev = e;
    e = next;
  }
 
  return e;
}
 
//根据Entry来删除HashMap中的值
final Entry<K,V> removeMapping(Object o) {
  if (size == 0 || !(o instanceof Map.Entry))
    return null;
 
  Map.Entry<K,V> entry = (Map.Entry<K,V>) o;
  Object key = entry.getKey();//第一步也是先获得该Entry中保存的key
  int hash = (key == null) ? 0 : hash(key);//接下来就和上面根据key删除Entry道理一样了
  int i = indexFor(hash, table.length);
  Entry<K,V> prev = table[i];
  Entry<K,V> e = prev;
 
  while (e != null) {
    Entry<K,V> next = e.next;
    if (e.hash == hash && e.equals(entry)) {
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
 
//清空HashMap中所有的Entry
public void clear() {
  modCount++;
  Arrays.fill(table, null);//将table中存储的Entry全部置为null
  size = 0;//size置为0
}
 
//判断HashMap中是否有key映射到指定的value
public boolean containsValue(Object value) {
  if (value == null)//如果value为空，调用下面特定的containsNullValue()方法
    return containsNullValue();
 
  Entry[] tab = table;//否则遍历链表中的每个Entry
  for (int i = 0; i < tab.length ; i++)
    for (Entry e = tab[i] ; e != null ; e = e.next)
      if (value.equals(e.value))//如果有Entry中的value与指定的value相等
        return true;//返回true
  return false;
}
 
//value为空时调用的方法
private boolean containsNullValue() {
  Entry[] tab = table;
  for (int i = 0; i < tab.length ; i++)
    for (Entry e = tab[i] ; e != null ; e = e.next)
      if (e.value == null)//与上面的不为空时大同小异
        return true;
  return false;
}
 
//克隆HashMap实例，这里是浅复制，并没有复制键和值的本身
public Object clone() {
  HashMap<K,V> result = null;
  try {
    result = (HashMap<K,V>)super.clone();
  } catch (CloneNotSupportedException e) {
    // assert false;
  }
  if (result.table != EMPTY_TABLE) {
    result.inflateTable(Math.min(
      (int) Math.min(
        size * Math.min(1 / loadFactor, 4.0f),
        // we have limits...
        HashMap.MAXIMUM_CAPACITY),
       table.length));
  }
  result.entrySet = null;
  result.modCount = 0;
  result.size = 0;
  result.init();
  result.putAllForCreate(this);
 
  return result;
}
/********************** 下面跟Iterator有关了 ************************/
//hashIterator实现了Iterator接口
private abstract class HashIterator<E> implements Iterator<E> {
  Entry<K,V> next;        // 下一个Entry
  int expectedModCount;   // 用于fail-fast机制
  int index;              // 当前索引
  Entry<K,V> current;     //当前的Entry
 
  HashIterator() {
    expectedModCount = modCount;//保存modCount用于fail-fast机制
    if (size > 0) { // advance to first entry
      Entry[] t = table;
      while (index < t.length && (next = t[index++]) == null)
        ;
    }
  }
  //判断有没有下一个Entry
  public final boolean hasNext() {
    return next != null;
  }
  //获得下一个Entry
  final Entry<K,V> nextEntry() {
    if (modCount != expectedModCount)//在迭代的过程中发现被修改了，就会抛出异常
      throw new ConcurrentModificationException();//即fail-fast
    Entry<K,V> e = next;
    if (e == null) //没有就抛出异常
      throw new NoSuchElementException();
 
    if ((next = e.next) == null) {
      Entry[] t = table;
      while (index < t.length && (next = t[index++]) == null)
        ;
    }
    current = e;
    return e;
  }
 
  public void remove() {//删除
    if (current == null)
      throw new IllegalStateException();
    if (modCount != expectedModCount)
      throw new ConcurrentModificationException();
    Object k = current.key;
    current = null;
    HashMap.this.removeEntryForKey(k);
    expectedModCount = modCount;
  }
}
//内部class ValueIterator迭代器，它修改了next方法
private final class ValueIterator extends HashIterator<V> {
  public V next() {
    return nextEntry().value;
  }
}
//内部class KeyIterator迭代器，它修改了next方法
private final class KeyIterator extends HashIterator<K> {
  public K next() {
    return nextEntry().getKey();
  }
}
//内部class EntryIterator迭代器，它修改了next方法
private final class EntryIterator extends HashIterator<Map.Entry<K,V>> {
  public Map.Entry<K,V> next() {
    return nextEntry();
  }
}
 
//定义上面三个对应的Iterator方法
Iterator<K> newKeyIterator()   {
  return new KeyIterator();
}
Iterator<V> newValueIterator()   {
  return new ValueIterator();
}
Iterator<Map.Entry<K,V>> newEntryIterator()   {
  return new EntryIterator();
}
 
private transient Set<Map.Entry<K,V>> entrySet = null;
/** 
 * 返回此映射中所包含的键的 Set 视图。 
 * 该 set 受映射的支持，所以对映射的更改将反映在该 set 中， 
 * 反之亦然。如果在对 set 进行迭代的同时修改了映射（通过迭代器自己的 remove 操作除外）， 
 * 则迭代结果是不确定的。该 set 支持元素的移除，通过  
 * Iterator.remove、Set.remove、removeAll、retainAll 和 clear 操作 
 * 可从该映射中移除相应的映射关系。它不支持 add 或 addAll 操作。 
 */ 
public Set<K> keySet() {
  Set<K> ks = keySet;
  return (ks != null ? ks : (keySet = new KeySet()));
}
 
private final class KeySet extends AbstractSet<K> {
  public Iterator<K> iterator() {
    return newKeyIterator();
  }
  public int size() {
    return size;
  }
  public boolean contains(Object o) {
    return containsKey(o);
  }
  public boolean remove(Object o) {
    return HashMap.this.removeEntryForKey(o) != null;
  }
  public void clear() {
    HashMap.this.clear();
  }
}
 
/** 
 * 返回此映射所包含的值的 Collection 视图。 
 * 该 collection 受映射的支持，所以对映射的更改将反映在该 collection 中， 
 * 反之亦然。如果在对 collection 进行迭代的同时修改了映射（通过迭代器自己的 remove 操作除外）， 
 * 则迭代结果是不确定的。该 collection 支持元素的移除， 
 * 通过 Iterator.remove、Collection.remove、removeAll、retainAll 和 clear 操作 
 * 可从该映射中移除相应的映射关系。它不支持 add 或 addAll 操作。 
 */  
public Collection<V> values() {
  Collection<V> vs = values;
  return (vs != null ? vs : (values = new Values()));
}
 
private final class Values extends AbstractCollection<V> {
  public Iterator<V> iterator() {
    return newValueIterator();
  }
  public int size() {
    return size;
  }
  public boolean contains(Object o) {
    return containsValue(o);
  }
  public void clear() {
    HashMap.this.clear();
  }
}
 
/** 
 * 返回此映射所包含的映射关系的 Set 视图。  
 * 该 set 受映射支持，所以对映射的更改将反映在此 set 中， 
 * 反之亦然。如果在对 set 进行迭代的同时修改了映射 
 * （通过迭代器自己的 remove 操作，或者通过在该迭代器返回的映射项上执行 setValue 操作除外）， 
 * 则迭代结果是不确定的。该 set 支持元素的移除， 
 * 通过 Iterator.remove、Set.remove、removeAll、retainAll 和 clear 操作 
 * 可从该映射中移除相应的映射关系。它不支持 add 或 addAll 操作。 
 */  
public Set<Map.Entry<K,V>> entrySet() {
  return entrySet0();
}
 
private Set<Map.Entry<K,V>> entrySet0() {
  Set<Map.Entry<K,V>> es = entrySet;
  return es != null ? es : (entrySet = new EntrySet());
}
 
private final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
  public Iterator<Map.Entry<K,V>> iterator() {
    return newEntryIterator();
  }
  public boolean contains(Object o) {
    if (!(o instanceof Map.Entry))
      return false;
    Map.Entry<K,V> e = (Map.Entry<K,V>) o;
    Entry<K,V> candidate = getEntry(e.getKey());
    return candidate != null && candidate.equals(e);
  }
  public boolean remove(Object o) {
    return removeMapping(o) != null;
  }
  public int size() {
    return size;
  }
  public void clear() {
    HashMap.this.clear();
  }
}
 
/**************************　序列化　*****************************/
private void writeObject(java.io.ObjectOutputStream s)
  throws IOException
{
  // Write out the threshold, loadfactor, and any hidden stuff
  s.defaultWriteObject();
 
  //将table.length写入流
  if (table==EMPTY_TABLE) {
    s.writeInt(roundUpToPowerOf2(threshold));
  } else {
     s.writeInt(table.length);
  }
 
  //将size写入流
  s.writeInt(size);
 
  //这里之所以不直接将table写出，而是分开写里面保存Entry的key和value的原因是：
  //table数组定义为了transient，也就是说在进行序列化时，并不包含该成员。
  //为什么将其设置为transient呢？因为Object.hashCode方法对于一个类的两个实例返回的是不同的哈希值。
  //即我们在机器A上算出对象A的哈希值与索引，然后把它插入到HashMap中，然后把该HashMap序列化后，
  //在机器B上重新算对象的哈希值与索引，这与机器A上算出的是不一样的，
  //所以我们在机器B上get对象A时，会得到错误的结果。
  //所以我们分开序列化key和value
  if (size > 0) {
    for(Map.Entry<K,V> e : entrySet0()) {
      s.writeObject(e.getKey());
      s.writeObject(e.getValue());
    }
  }
}
 
private static final long serialVersionUID = 362498820763181265L;
 
private void readObject(java.io.ObjectInputStream s)
   throws IOException, ClassNotFoundException
{
  // Read in the threshold (ignored), loadfactor, and any hidden stuff
  s.defaultReadObject();
  if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
    throw new InvalidObjectException("Illegal load factor: " +
                       loadFactor);
  }
 
  // set other fields that need values
  table = (Entry<K,V>[]) EMPTY_TABLE;
 
  // Read in number of buckets
  s.readInt(); // ignored.
 
  // Read number of mappings
  int mappings = s.readInt();
  if (mappings < 0)
    throw new InvalidObjectException("Illegal mappings count: " +
                       mappings);
 
  // capacity chosen by number of mappings and desired load (if >= 0.25)
  int capacity = (int) Math.min(
        mappings * Math.min(1 / loadFactor, 4.0f),
        // we have limits...
        HashMap.MAXIMUM_CAPACITY);
 
  // allocate the bucket array;
  if (mappings > 0) {
    inflateTable(capacity);
  } else {
    threshold = capacity;
  }
 
  init();  // Give subclass a chance to do its thing.
 
  // Read the keys and values, and put the mappings in the HashMap
  for (int i = 0; i < mappings; i++) {
    K key = (K) s.readObject();
    V value = (V) s.readObject();
    putForCreate(key, value);
  }
}
 
// These methods are used when serializing HashSets
int   capacity()     { return table.length; }
float loadFactor()   { return loadFactor;   }
```