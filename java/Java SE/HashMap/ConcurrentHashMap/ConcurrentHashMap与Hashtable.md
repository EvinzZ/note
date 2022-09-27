

# ConcurrnetHashMap VS HashTable

1. Hashtable与ConcurrentHashMap都是线程安全的Map集合
2. Hashtabe并发度低，整个Hashtable对应一把锁，同一时刻，只能有一个线程操作它
3. 1.8之前ConcurrentHashMap使用了Segment+数组+链表的结构，每个Segment对应一把锁，如果多个线程访问不同的Segment，则不会冲突
4. 1.8开始`ConcurrentHashMap`将数组的每个头节点作为锁，如果多个线程访问的头节点不同，则不会冲突