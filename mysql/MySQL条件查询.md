## 语法

```sql
select 查询列表 from 表明 where 条件;
```

## MySQL执行流程

```sql
select 字段（步骤3）from 表名（步骤1）where 条件（步骤2）
```

## 运算符分类

### 按条件表达式筛选

1. `>` 大于
2. `<` 小于
3. `=` 等于
4. `<>` 或者 `!=` 不等于
5. `>=` 大于等于
6. `<=` 小于等于

### 按逻辑表示式

1. `&&` 或者 `and` ：且
2. `||` 或者 `or` ：或
3. `!` 或者 `not`：非

## 模糊查询：`like`

```sql
select * from tab where name like '%a%'; # %为通配符
```

通常与通配符一起使用

通配符：

    1.`%` ：任意多个字符，包含0个字符
    
    2.`_` ：任意单个字符

自定义转义符

```sql
select * from where name like '_$_%' ESCAPE '$'; // 此处$为自定义的转义符
```

## 区间查询：`between and`

```sql
select * from where id between 1 and 10;
```

注意：

1. 使用between and可以提高语句的简洁度

2. 包含临界值

3. 两个临界值不要调换顺序

4. not between and 是不在区间内

## 判断某字段的值是否属于in列表中的某一项：`in`

```sql
select * from tab where name in ('a', 'b', 'c');
```

特点：

1. 使用in提高语句简洁度

2. in列表的值类型必须一直或者兼容

## 判断是否为空：`is null`, `is not null`

```sql
select * from tab where id is null;
```

注意：

1. `=` 或者 `<>` 不能用于判断null值
2. `is null` 或 `is not null `可以判断null

## 安全等于：`<==>`

可以判断null和普通类型

## 排序

### 语法

```sql
select 查询列表
from 表
[where 筛选条件]
order by 排序列表 asc | desc;
```

### 倒序

```sql
select * from tab order by id desc;
```

### 正序

```sql
select * from tab order by id asc;
```

特点

* asc代表的是升序，desc代表的是降序，默认不写是升序
* order by子句中可以支持单个字段、多个字段、表达式、函数、别名
* order by子句一般是放在查询语句的最后面，limit子句除外