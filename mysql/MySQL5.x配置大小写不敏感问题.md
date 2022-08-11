# MySQL5.x 配置大小写不敏感问题

1、查看数据库是否区分大小写：

```sql
show variables like "%case%";
```

lower_case_table_names =0时为区分大小写。

2、修改

```bash
vim /etc/my.cnf
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
lower_case_table_name=1
```

3、重启MySQL

```bash
systemctl restart mysqld
```