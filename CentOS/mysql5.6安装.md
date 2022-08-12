# mysql5.6安装

## 1.centOS7卸载自带maridb，安装mysql5.6

### 1.1.列出安装的包

```bash
rpm -qa | grep mariadb
```

### 1.2.卸载包

```bash
rpm -e --nodeps mariadb-libs-5.5.41-2.el7_0.x86_64
```

## 2.安装mysql

### 2.1. 查看当前可用的mysql安装资源

```bash
yum repolist enabled | grep "mysql.*-community.*"
```

### 2.2. 安装

```bash
 yum -y install mysql-community-server
```

### 2.3. 配置开机自启

```bash
systemctl enable mysqld
```

### 2.4. 启动服务

```bash
systemctl start mysqld
```

### 2.5. 配置mysql（密码等）

```bash
[root@localhost ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MySQL to secure it, we'll need the current
password for the root user.  If you've just installed MySQL, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MySQL
root user without the proper authorisation.

Set root password? [Y/n] y[设置root用户密码]
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y[删除匿名用户]
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y[禁止root远程登录]
 ... Success!

By default, MySQL comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y[删除test数据库]
 - Dropping test database...
ERROR 1008 (HY000) at line 1: Can't drop database 'test'; database doesn't exist
 ... Failed!  Not critical, keep moving...
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y[刷新权限]
 ... Success!




All done!  If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!


Cleaning up...

```

### 2.6. 配置外部访问

```bash
登录MySQL然后操作
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你想使用的外部连接的密码' WITH GRANT OPTION;   // 开启外部连接
flush privileges;
```

## 3.卸载mysql

### 3.1. 查找已经安装的mysql

```bash
rpm -qa | grep -i mysql

mysql-community-common-5.6.48-2.el7.x86_64
mysql-community-libs-5.6.48-2.el7.x86_64
mysql-community-release-el7-5.noarch
mysql-community-client-5.6.48-2.el7.x86_64
mysql-community-server-5.6.48-2.el7.x86_64
```

### 3.2. 卸载

```bash
yum -y remove mysql-*
```

### 3.3. 删除目录

查找mysql的一些目录，把所有出现的目录统统删除，可以使用rm -rf 路径，删除时请注意，一旦删除无法恢复

```bash
find / -name mysql
```

### 3.4. 删除配置文件

```bash
rm -rf /etc/my.cnf
```

### 3.5. 删除默认密码

删除mysql的默认密码，如果不删除，以后安装mysql这个sercret中的默认密码不会变，使用其中的默认密码就可能会报类似`Access denied for user 'root@localhost'(using password:yes)`的错误。

```bash
rm -rf /root/.mysql_sercret
```

