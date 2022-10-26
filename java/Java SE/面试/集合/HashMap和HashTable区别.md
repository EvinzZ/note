# HashMap和HashTable区别

1. HashTable线程同步，HashMap非线程同步
2. HashTable不允许<键，值>有空值，HashMap允许<键，值>有空值
3. HashTable使用Enumeration，HashMap使用Iterator
4. HashTable中hash数组的默认大小是11，增加方式的old*2+1，HashMap中hash数组的默认大小是16，增长方式是2的指数倍。
5. hashtable继承于Dictionary类，hashmap继承自AbstractMap类