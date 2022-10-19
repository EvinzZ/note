# ThreadLocal

## 概念

ThreadLoal 变量，线程局部变量，同一个 ThreadLocal 所包含的对象，在不同的 Thread 中有不同的副本。这里有几点需要注意：

- 因为每个 Thread 内有自己的实例副本，且该副本只能由当前 Thread 使用。这是也是 ThreadLocal 命名的由来。
- 既然每个 Thread 有自己的实例副本，且其它 Thread 不可访问，那就不存在多线程间共享的问题。

ThreadLocal 提供了线程本地的实例。它与普通变量的区别在于，每个使用该变量的线程都会初始化一个完全独立的实例副本。ThreadLocal 变量通常被private static修饰。当一个线程结束时，它所使用的所有 ThreadLocal 相对的实例副本都可被回收。

总的来说，ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用，也即变量在线程间隔离而在方法或类间共享的场景。

总结：

1. 线程间资源隔离
2. 线程内资源共享

## 实现原理

首先 ThreadLocal 是一个泛型类，保证可以接受任何类型的对象。

因为一个线程内可以存在多个 ThreadLocal 对象，所以其实是 ThreadLocal 内部维护了一个 Map ，这个 Map 不是直接使用的 HashMap ，而是 ThreadLocal 实现的一个叫做 ThreadLocalMap 的静态内部类。而我们使用的 get()、set() 方法其实都是调用了这个ThreadLocalMap类对应的 get()、set() 方法。

例如下面的 set 方法：

```Java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

get方法：

```Java
public T get() {   
        Thread t = Thread.currentThread();   
        ThreadLocalMap map = getMap(t);   
        if (map != null)   
            return (T)map.get(this);   
  
        // Maps are constructed lazily.  if the map for this thread   
        // doesn't exist, create it, with this ThreadLocal and its   
        // initial value as its only entry.   
        T value = initialValue();   
        createMap(t, value);   
        return value;   
    }
```

createMap方法：

```Java
    void createMap(Thread t, T firstValue) {   
        t.threadLocals = new ThreadLocalMap(this, firstValue);   
    } 
```

ThreadLocalMap是个静态的内部类：

```Java
    static class ThreadLocalMap {   
    ........   
    }  
```

最终的变量是放在了当前线程的 `ThreadLocalMap` 中，并不是存在 ThreadLocal 上，ThreadLocal 可以理解为只是ThreadLocalMap的封装，传递了变量值。

总结：

其原理是，每个线程内有一个`ThreadLocalMap`类型的成员变量，用来存储资源对象

1. 调用`set()`方法，就是以`ThreadLocal`自己作为key，资源对象作为value，放入当前线程的`ThreadLocalMap`集合中
2. 调用`get()`方法，就是以`ThreadLocal`自己作为key，到当前线程中查找关联的资源值。
3. 调用`remove()`方法，就是以`ThreadLocal`自己作为key，移除当前线程关联的资源值。

## 内存泄漏问题

实际上 ThreadLocalMap 中使用的 key 为 ThreadLocal 的弱引用，弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

所以如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，这样一来 ThreadLocalMap中使用这个 ThreadLocal 的 key 也会被清理掉。但是，value 是强引用，不会被清理，这样一来就会出现 key 为 null 的 value。

ThreadLocalMap实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录。如果说会出现内存泄漏，那只有在出现了 key 为 null 的记录后，没有手动调用 remove() 方法，并且之后也不再调用 get()、set()、remove() 方法的情况下。

建议回收自定义的ThreadLocal变量，尤其在线程池场景下，线程经常会被复用，如果不清理自定义的 ThreadLocal变量，可能会影响后续业务逻辑和造成内存泄露等问题。 尽量在代理中使用try-finally块进行回收：

```Java
objectThreadLocal.set(userInfo); 
try {
    // ... 
} 
finally {
    objectThreadLocal.remove(); 
}
```

## 使用场景

如上文所述，ThreadLocal 适用于如下两种场景

- 每个线程需要有自己单独的实例
- 实例需要在多个方法中共享，但不希望被多线程共享

对于第一点，每个线程拥有自己实例，实现它的方式很多。例如可以在线程内部构建一个单独的实例。ThreadLoca 可以以非常方便的形式满足该需求。

对于第二点，可以在满足第一点（每个线程有自己的实例）的条件下，通过方法间引用传递的形式实现。ThreadLocal 使得代码耦合度更低，且实现更优雅。

1）存储用户Session

一个简单的用ThreadLocal来存储Session的例子：

```Java
private static final ThreadLocal threadSession = new ThreadLocal();

    public static Session getSession() throws InfrastructureException {
        Session s = (Session) threadSession.get();
        try {
            if (s == null) {
                s = getSessionFactory().openSession();
                threadSession.set(s);
            }
        } catch (HibernateException ex) {
            throw new InfrastructureException(ex);
        }
        return s;
    }
```

2）解决线程安全的问题

比如Java7中的SimpleDateFormat不是线程安全的，可以用ThreadLocal来解决这个问题：

```Java
public class DateUtil {
    private static ThreadLocal<SimpleDateFormat> format1 = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };

    public static String formatDate(Date date) {
        return format1.get().format(date);
    }
}
```

这里的DateUtil.formatDate()就是线程安全的了。(Java8里的 **`[java.time.format.DateTimeFormatter]`**`是线程安全的，Joda time里的DateTimeFormat也是线程安全的）。`

## **ThreadLocalRandom**

ThreadLocalRandom使用ThreadLocal的原理，让每个线程内持有一个本地的种子变量，该种子变量只有在使用随机数时候才会被初始化，多线程下计算新种子时候是根据自己线程内维护的种子变量进行更新，从而避免了竞争。

用法：

```Java
ThreadLocalRandom.current().nextInt(100)
```

## 总结

在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。 初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。 然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

1. 实际的通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals中的；
2. 为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象，因为每个线程中可有多个threadLocal变量，就像上面代码中的longLocal和stringLocal；
3. 在进行get之前，必须先set，否则会报空指针异常；如果想在get之前不需要调用set就能正常访问的话，必须重写initialValue()方法。 因为在上面的代码分析过程中，我们发现如果没有先set的话，即在map中查找不到对应的存储，则会通过调用setInitialValue方法返回i，而在setInitialValue方法中，有一个语句是T value = initialValue()， 而默认情况下，initialValue方法返回的是null。

## 为什么`ThreadLocalMap`中的key（即`ThreadLocal`）要设计为弱引用？

1. `Thread`可能需要长时间运行（如线程池中的线程），如果key不再使用，需要在内存不足（GC）时释放其占用的内存
2. 但GC仅是让key的内存释放，后续还要根据key是否为null来进一步释放值的内存，释放时机有
   1. 获取key发现null key
   1. 
   2. set key时，会使用启发式扫描，清除临近的null key，启发次数与元素个数，是否发现null key有关
   3. `remove`时（推荐），因为一般使用`ThreadLocal`时都把它作为静态变量，因此GC无法回收

## 谈一谈对ThreadLocal的理解

1. `ThreadLocal`可以实现【资源对象】的线程隔离，让每个线程各用各的【资源对象】，避免争用引发的线程安全问题
2. `ThreadLocal`同时实现了线程内的资源共享

3. 其原理是，每个线程内有一个`ThreadLocalMap`类型的成员变量，用来存储资源对象
   1. 调用 set 方法，就是以`ThreadLocal`自己作为 key，资源对象作为 value，放入当前对象的`ThreadLocalMap`集合中
   2. 调用 get 方法，就是以`ThreadLocal`自己作为 key ，到当前线程中查找关联的资源值
   3. 调用 remove 方法，就是以`ThreadLocal`自己作为key，移除当前线程关联的资源值

