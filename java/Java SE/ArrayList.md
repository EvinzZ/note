# ArrayList

## 扩容机制

- `ArrayList()`会使用长度为零的数组
- `ArrayList(int initialCapacity)`会使用指定容量的数组
- `public ArrayList(Collection<? extends E> c>)`会使用c的大小作为数组容量
- `add(Object o)`首次扩容为10，再次扩容为上次容量的1.5倍
- `addAll(Collection c)`没有元素时，扩容为`Math.max(10, 实际元素个数)`，有元素时为`Math.max(原容量1.5倍, 实际元素个数)`
- 扩容计算方式为原容量+原容量右移1位 `原容量 >> 1` + 原容量

## Iterator的fail-fast与fail-safe机制

fail-fast：

一旦发现遍历的同时其他人来修改，则立刻抛异常

fail-safe：

发现遍历的同时其他人来修改，应当能有应对策略，例如牺牲一致性来让整个遍历运行完成

- ArrayList是fail-fast的典型代表，遍历的同时不能修改，尽快失败
- CopyOnWriteArrayList是fail-safe的典型代表，遍历的同时可以修改，原理是读写分离