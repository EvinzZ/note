# volatile

## 1.**概念**

如果一个字段被声明程volatile，java线程内存模型确保所有线程看到这个变量的值是一致的。

volatile是轻量级的synchronized，他在多处理器开发中保证了共享变量的“可见性”。

可见性 ： 可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。

如果volatile变量修饰符使用恰当的话，它比synchronized的使用和执行成本更低，因为它不会引起线程上下文的切换和调度。

volatile关键字，是一个变量在多个线程间可见，A，B线程都用到一个变量，java默认是A线程中保留一份copy，这样如果B线程修改了该变量，则A线程未必知道，使用volatile关键字，会让所有线程都会读到变量的修改值。（volatile boolean raa = true;）

volatile可以用来修改字段（成员变量），就是告知程序任何对该变量的访问均需要从共享内存中获取，而对它的改变必须同步刷新回共享内存，它能保证所有线程对变量访问的可见性。

## 2.**volatile 和 synchronized的区别**

volatile只保证了可见性，synchronized既保证了可见性，又保证了原子性。synchronized效率低。

## 3.**CPU的术语定义**

1. 内存屏障 ： 是一组处理器指令，用于实现对内存操作的顺序限制
2. 缓冲行 ： CPU高速缓存中可以分配的最小存储单位。处理器填写缓存行时会加载整个缓存行，现代CPU需要执行几百次CPU指令
3. 原子操作 ： 不可中断的一个或一些列操作
4. 缓存行填充 ： 当处理器识别到从内存中读取操作数是可缓存的，处理器读取整个高速缓存行到适当的缓存（L1、L2、L3的或所有）
5. 缓存命中 ： 如果进行高速缓存行填充操作的内存位置仍然是下次处理器访问的地址时，处理器从缓存中读取操作数，而不是从内存读取。
6. 写命中 ： 当处理器将操作数写回到一个内存缓存的区域时，它首先会检查这个缓存的内存地址是否在缓存行中，如果存在一个有效地缓存行，则处理器将这个操作数写回到缓存，而不是写回到内存，这个操作被称为写命中。
7. 写缺失 ： 一个有效地缓存行被写入到不存在的内存区域

## 4.**volatile实现原理**

有volatile变量修饰的共享变量进行写操作的时候回多出第二行汇编代码， 0x01a3de1d: movb $0x0,0x1104800(%est);0x01a3de24: **lock** addl $0x0,(%esp);

Lock前缀的指令在多核处理器下会引发了两件事情：

1、将当前处理器缓存行的数据写回到系统内存

2、这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效。

为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后在进行操作，但操作完不知道何时会写到内存，如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

两条原则：

1. Lock前缀指令会引起处理器缓存写回到内存中
2. 一个处理器的缓存回写到内存会导致其他处理器的缓存无效

## 5.**注意**

- 过多的使用volatile是不必要的，因为它会降低程序执行的效率。
- volatile并不能保证多个线程共同修改running变量时所带来的不一致问题，也就是说volatile不能替代synchronized

## 6.可见性案例

```Java
// 一个线程修改了变量，两一个线程没有读到修改后的最新值
public class Demob {
    static boolean stop = false;

    public static void main(String[] args) {
        new Thread(()->{
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            stop = true;
            System.out.println("main() run.");
        }).start();
        fun();
    }

    static void fun() {
        int i = 0;
        while (!stop) {
            i++;
        }
        System.out.println("fun() run." + i);
    }
}
```

## 7.面试题

### 7.1.`volatile`能否保证线程安全

线程安全要考虑三个方面：可见性、有序性、原子性

1. 可见性：一个线程对共享变量修改，另一个线程能看到最新的结果
2. 有序性：一个线程内代码按编写顺序执行
3. 原子性：一个线程内多行代码以一个整体运行，期间不能有其他线程的代码插队

`volatile`能够保证共享变量的可见性与有序性，但不能保证原子性