## jvisualvm工具

## arthas

## JVM参数选项

| 参数                            | 作用                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| -Xms                            | 初始堆大小。如：-Xms256m                                     |
| -Xmx                            | 最大堆大小。如：-Xmx512m                                     |
| -Xmn                            | 新生代大小。通常为 Xmx 的 1/3 或 1/4。新生代 = Eden + 2 个 Survivor 空间。实际可用空间为 = Eden + 1 个 Survivor，即 90% |
| -Xss                            | JDK1.5+ 每个线程堆栈大小为 1M，一般来说如果栈不是很深的话， 1M 是绝对够用了的。 |
| -XX:NewRatio                    | 新生代与老年代的比例，如 –XX:NewRatio=2，则新生代占整个堆空间的1/3，老年代占2/3 |
| -XX:SurvivorRatio               | 新生代中 Eden 与 Survivor 的比值。默认值为 8。即 Eden 占新生代空间的 8/10，另外两个 Survivor 各占 1/10 |
| -XX:PermSize                    | 永久代(方法区)的初始大小                                     |
| -XX:MaxPermSize                 | 永久代(方法区)的最大值                                       |
| -XX:+PrintGCDetails             | [打印](http://cpro.baidu.com/cpro/ui/uijs.php?rs=1&u=http%3A%2F%2Fwww.th7.cn%2FProgram%2Fjava%2F201409%2F276272.shtml&p=baidu&c=news&n=10&t=tpclicked3_hc&q=smileking_cpr&k=%B4%F2%D3%A1&k0=java&kdi0=8&k1=%B4%F2%D3%A1&kdi1=1&sid=dd8622b3256b78a7&ch=0&tu=u1682280&jk=d0ec83cbcc6c0312&cf=29&fv=11&stid=9&urlid=0&luki=2&seller_id=1&di=128) GC 信息 |
| -XX:+HeapDumpOnOutOfMemoryError | 让虚拟机在发生内存溢出时 Dump 出当前的内存堆转储快照，以便分析用 |