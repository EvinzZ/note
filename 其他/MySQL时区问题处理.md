# MySQL时区问题

## 方法一

### 查看时区

```SQL
show variables like "%time_zone%";
set global time_zone = '+8:00'; ##修改mysql全局时区为北京时间，即我们所在的东8区
set time_zone = '+8:00'; ##修改当前会话时区
flush privileges; #立即生效

select now(); -- 查询当前时间是否和系统时间一致
```

如果代码插入的时间还是不对，看看连接语句

```YAML
url: jdbc:mysql://192.168.1.107:3306/datacollection_dev?characterEncoding=utf8&serverTimezone=Asia/Shanghai&useSSL=false
```

后面的时区写成上海

## 方法二

```SQL
linux

# vim /etc/my.cnf ##在[mysqld]区域中加上
default-time_zone = '+8:00'
# /etc/init.d/mysqld restart ##重启mysql使新时区生效
windows
修改配置文件my.ini

路径C:\ProgramData\MySQL\MySQL Server 5.7

ProgramData是隐藏文件夹

在[mysqld]下添加

#设置默认时区
default-time-zone='+08:00'
以管理员身份在cmd中重启mysql

net stop mysql
net start mysql
如果报错

服务名无效。 请键入 NET HELPMSG 2185 以获得更多的帮助。
为服务名错误 请在服务中查看服务名

windows+r， 打开运行，输入services.msc, 找到mysql的服务，

查看服务名为mysql57，

net stop mysql57
net start mysql57
```