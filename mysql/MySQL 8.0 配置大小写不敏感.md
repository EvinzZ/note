# MySQL 8.0 配置大小写不敏感

1.停止服务

```bash
systemctl stop mysqld
```

2.修改配置

```bash
vim /etc/my.cnf

[mysqld]
lower_case_table_names = 1
```

3.删文件

```bash
rm -rf /var/lib/mysql/*
```

4.重启

```bash
systemctl start mysqld
```