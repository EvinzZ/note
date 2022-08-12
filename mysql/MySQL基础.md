## 数据库存储数据的特点

* 将数据放到表中，表再放到库中
* 一个数据库中可以有多个表，每个表都有一个名字，用来标识自己。表名具有唯一性。
* 表具有一些特性，这些特性定义了数据在表中如何存储，类似java中“类”的设计。
* 表由列组成，我们也成为字段。所有表都是有一个或多个列组成的，每一列类似java中的“属性”
* 表中的数据是按行存储的，每一行类似于java中的“对象”。

## 语句执行顺序

```sql
select 查询列表         7
from 表                1
[join type join 表2    2
on 连接条件             3
where 筛选条件          4
group by 分组字段       5
having 分组后的筛选      6
order by 排序后的字段    8
limit [offset,] size   9

offset:要显示条目的起始索引
size  :要显示的条目个数

注意：limit语句总是放在查询语句的最后
```

## 常用命令

### 显示全部数据库

```sql
SHOW DATABASES;
```

### 切换数据库

```sql
USE test;
```

### 显示全部表

```sql
SHOW TABLES;
或者
SHOW TABLES FROM test;
```

### 显示所在库

```sql
SELECT database();
```

### 创建表

```sql
CREATE TABLE aa(
    id int,
    name varchar(20)
);
```

### 查看表结构

```sql
DESC aa;
```

### 查询MySQL版本

```sql
SELECT version();
或者
mysql -v
```

## 语法规范

* 不区分大小写，但建议关键字大写，表明，列名小写。
* 每条命令最好用分号结尾。
* 根据需要进行缩进换行

### 注释

```sql
单行注释：#
单行注释：-- 注释文字（--后面有一个空格）
多行注释：/*  */
```

## 查询

```sql
select 查询列表 from 表名;
```

## 起别名

```sql
select 字段 AS x from table;
方式二： select 字段 别名 from table;
```

好处

1. 便于理解
2. 如果要查询的字段有重名的情况，使用别名可以区分开

## 去重

```sql
select distinct id from tab;
```

## MySQL中的+号

作用：运算符

* 两个操作数都为数值型，则做加法运算
* 只要其中一方为字符型，试图将字符型数值转换成数值型
  * 如果转换成功，则继续做加法运算
  * 如果转换失败，则将字符型数值转换为0
* 只要其中一方为null，则结果肯定为null。select null + 值;结果都为Null