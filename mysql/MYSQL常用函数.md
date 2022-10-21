## mysql函数

函数

函数可以嵌套使用

**concat函数**

功能：拼接字符

​                select concat(字符1, 字符2, 字符3,.....);              

**ifnull函数**

功能：判断某字段或表达式是否为null，如果为null，返回指定的值，否则返回原本的值

​                select ifnull(commission_pct, 0) from tab;              

**isnull函数**

功能：判断某字段是否为null，如果为null，返回1，否则返回0

​                select isnull(id) from tab;              

**length()函数**

功能：获取参数值的字节个数

​                select length('dddddd');              

**upper()函数**

功能：转大写

​                select upper('aaaa');              

**lower()函数**

功能：转小写

​                select lower('AAAAA');              

**substr()**

功能：截取字符

​                select substr('aaaaaa', 2); # 截取从指定索引处后面所有字符 select substr('aaaaa', 2,4); # 截取从指定索引处指定字符长度的字符              

注意：索引从1开始

**instr()**

功能：返回子串第一次出现的索引，如果找不到返回0

​                select instr('ggshgfsfgsdfklsdjfksdf', 'sh');              

**trim()**

功能：去除前后字符

​                select trim('             asd             '); # 去除字符前后空格 select trim('a' from 'aaaaaaaaaaaaaa好aaaaaaa好aaaaaaaaa'); # 去除前后a              

**lpad()**

功能：用指定的字符实现左填充指定长度

​                select lpad('哈哈哈', 10, '*');              

**rpab()**

功能：用指定的字符实现右填充指定长度

​                select rpad('哈哈哈', 10, '*');              

**replace()**

功能：替换

​                select replace('我的天哪', '哪', '啊');              

\##########数学函数

**round()**

功能：四舍五入

​                select round(1.65); select round(1.567, 2); # 四舍五入后保留两位小数              

**ceil()**

功能：向上取整，返回>=该整数的最小整数

​                select ceil(1.22);              

**floor()**

功能：向下取整，返回<=该参数的最大整数

​                select floor(9.99);              

**mod()**

功能：取余

注意：如果是负数进行取余，那么结果的正负和被除数的正负相同。

​                select mod(10, 3);              

\- 日期函数 -

**now()**

功能：返回当前系统日期+时间

​                select now();              

**curdate()**

功能：返回当前系统日期，不包含时间

​                select curdate();              

**curtime()**

功能：返回当前时间，不包含日期

​                select curtime();              

**获取指定的部分，年、月、日、时、分、秒**

​                select year(now()); # 年 select month(now()); # 月 select day(now()); # 日 select hour(now()); # 时 select minute(now()); # 分 select second(now()); # 秒 select monthname(now()); # 英文的月              

**str_to_date()**

功能：将日期格式的字符转换成指定格式的日期

​                select str_to_date('9-13-1999', '%m-%d-%Y');              

**date_format()**

功能：将日期转换成字符

​                select date_format('2018/6/6', '%Y年%m月%d日');              

**日期字符格式表**

| 序号 | 格式符 | 功能                |
| ---- | ------ | ------------------- |
| 1    | %Y     | 四位的年份          |
| 2    | %y     | 2位的年份           |
| 3    | %m     | 月份(01,02...11,12) |
| 4    | %c     | 月份(1,2,...11,12)  |
| 5    | %d     | 日(01,02....)       |
| 6    | %H     | 小时（24小时制）    |
| 7    | %h     | 小时（12小时制）    |
| 8    | %i     | 分钟(00,01....59)   |
| 9    | %s     | 秒(00, 01,....59)   |

\- 其他函数 -

**返回当前MySQL版本**

​                select version();              

**返回当前库**

​                select database();              

**返回当前用户**

​                select user();              

**流程控制函数**

**if**

​                select if(10 > 5, 'Yes', 'No'); select id,if(id is null, 'Y', 'N') as other from tab;              

**case**

​                case 要判断的字段或表达式 when 常量1 then 要显示的值1或语句1; when 常量2 then 要显示的值2或语句2; .. else 要显示的值n或语句n; end              

**case第二种用法，类似if else**

​                case when 条件1 then 要显示的值1或语句1 when 条件2 then 要显示的值2或语句2 .. else 要显示的值n或语句n end              

**分组函数**

**sum：求和**

​                select sum(id) from tab;              

参数类型：数值型

**AVG：平均数**

​                select avg(id) from tab;              

参数类型：数值型

**min：最小值**

​                select min(id) from tab;              

参数类型：字符型、数值型、日期型、可以排序的都行

**max：最大值**

​                select max(id) from tab;              

参数类型：字符型、数值型、日期型、可以排序的都行

**count：计算个数**

​                select count(id) from tab;              

参数类型：任何类型

注意：MyISAM存储引擎，count（*）效率最高

InnoDB存储引擎，count(*)和count(1)效率>count(字段)

**注意**

- sum(),avg()做运算时，如果字段中有null，那么会忽略null值，因为null+任何值都等于null。avg运算时，是非Null的值相加除去非null的值的总个数，null被忽略了。
- count()计算非空的值的个数

**和distinct搭配**

​                select sum(distinct id) from tab;              

**count()函数的详细介绍**

​                select count(id) from tab;  select count(*) from tab; // 统计行数 select count(1) from tab; // 统计行数              

效率：

MYISAM存储引擎下，count(*)的效率高。

INNODB存储引擎下，count(*)和count(1)的效率差不多，比Count(字段)要高一些。

注意：和分组函数一同查询的字段要求是group by后的字段

**DATEDIFF**

计算两个日期相差天数，第一个参数减去第二个参数

​                select datediff('2011-11-11, '2011-9-11');              