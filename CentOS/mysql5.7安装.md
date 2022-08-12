# mysql5.7安装

1、更换yum源

2、下载

​                `wget dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm              `

3、安装

​                `yum install mysql-community-release-el7-5.noarch.rpm -y           `   

4、修改 `/etc/yum.repos.d/mysql-community.repo`文件，修改下载的MySQL版本

```properties
[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://repo.mysql.com/yum/mysql-tools-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Enable to use MySQL 5.5
[mysql55-community]
name=MySQL 5.5 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.5-community/el/6/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Enable to use MySQL 5.6
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/
enabled=0       #####此处改为0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Note: MySQL 5.7 is currently in development. For use at your own risk.
# Please read with sub pages: https://dev.mysql.com/doc/relnotes/mysql/5.7/en/
[mysql57-community-dmr]
name=MySQL 5.7 Community Server Development Milestone Release
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1          #####此处改为1
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
~                                             
```

5、

​               ` yum install mysql-community-server -y          `    

注意：如果出现CentOS7yum安装mysql+需要：libsasl2.so.2()(64bit)

```properties
修改vim /etc/yum.repos.d/mysql-community.repo 源文件

[mysql57-community]

name=MySQL 5.7 Community Server

## baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/   

baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/  ### 这里改为7

enabled=1

gpgcheck=0  ### 这里改为0

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql    
```

6、最好更新一下yum

​                `yum update -y              `

7、启动mysql服务

​               ` systemctl start mysqld             ` 

8、设置开机自启

​              `  systemctl enable mysqld              `

9、获取mysql的临时密码

​               ` grep "password" /var/log/mysqld.log         `      

10、登录后修改密码

  ` set global validate_password_policy=0; ` 

  `set global validate_password_length=1; `

  `ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';       `       

11、修改远程访问权限

​       ` GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION; ` 

​       `flush privileges;       `       

12、设置字符集为utf-8，/etc/my.cnf

```properties
在[mysqld]部分添加：
default-storage-engine=INNODB
character-set-server=utf8
collation-server=utf8_general_ci
在文件末尾新增[client]端，并在[client]端添加：
default-character-set=utf8
```

