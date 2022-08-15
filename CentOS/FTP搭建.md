## 一、安装VSFTPD

1.为了方便后续操作，现将用户切换到root用户   su -

2.查看是否已经安装vsftpd

方法一：

```Bash
[root@localhost ~]# rpm -q vsftpd

vsftpd-3.0.2-22.el7.x86_64
```

方法二：

```Bash
[root@localhost ~]# vsftpd -v

yum安装vsftpd

[root@localhost ~]# yum -y install vsftpd
```

![img](../images/20180815165541219)

3.安装完成后,查看位置

[root@localhost ~]# whereis vsftpd

vsftpd: /usr/sbin/vsftpd /etc/vsftpd /usr/share/man/man8/vsftpd.8.gz

4.直接启动VSFTP服务

[root@localhost ~]# systemctl start vsftpd.service

[root@localhost ~]#

5.查看是否启动成功

[root@localhost ~]# netstat -npal|grep vsftpd

tcp6    0   0 :::21     :::*     LISTEN      4432/vsftpd

可以看到服务已经启动，端口为21，pid为4432

6.关闭SELinux限制，添加防火墙白名单

①设置关闭SELinux对ftp的限制

[root@localhost ~]# getsebool -a | grep ftp

ftp_home_dir --> on

ftpd_anon_write --> off

ftpd_connect_all_unreserved --> off

ftpd_connect_db --> off

ftpd_full_access --> on

ftpd_use_cifs --> off

ftpd_use_fusefs --> off

ftpd_use_nfs --> off

ftpd_use_passive_mode --> off

httpd_can_connect_ftp --> off

httpd_enable_ftp_server --> off

sftpd_anon_write --> off

sftpd_enable_homedirs --> off

sftpd_full_access --> off

sftpd_write_ssh_home --> off

tftp_anon_write --> off

tftp_home_dir --> off

[root@localhost ~]# setsebool -P ftpd_full_access on

②将ftp加入防火墙白名单

firewall-cmd --permanent --zone=public --add-service=ftp

firewall-cmd --reload

查看防火墙状态：firewall-cmd --list-all

## 配置修改

1.修改配置文件

[root@localhost ~]# cd /etc/vsftpd/

[root@localhost vsftpd]# vim vsftpd.conf

不允许匿名访问(不登录默认访问某目录/var/ftp)

anonymous_enable=NO

允许ascii文件上传和下载

ascii_upload_enable=YES

ascii_download_enable=YES

将用户限制在为其配置的主目录

chroot_local_user=YES

chroot_list_enable=YES

chroot_list_file=/etc/vsftpd/chroot_list

## 匿名登录

匿名登陆 如果设置anonymous_enable=NO表示可以匿名登陆，保存后重新启动vsftp服务systemctl restart vsftpd.service），即可以匿名登陆ftp服务（ftp ipaddr），密码是空，对应目录是/var/ftp，如果你在配置里面配置了anonymous_enable=NO，匿名就无法登录。

## 多用户配置

1.首先需要在vsftp.conf添加如下配置

\#设置启用虚拟用户功能

guest_enable=YES

\#制定宿主用户名(我们后续需要为我们的系统增加该用户)

guest_username=ftpuser

\#制定虚拟用户配置文件放置文件夹(需要我们自己建立)

user_config_dir=/etc/vsftpd/vuser_conf

\#允许写

allow_writeable_chroot=YES

2.创建宿主用户

\#创建宿主主文件夹

cd /home

mkdir vsftpd

# 创建用户 ftpuser 指定 `/home/vsftpd` 目录

# -s /sbin/nologin ftpuser 表示不允许该用户通过命令行方式登录

useradd -g root -M -d /home/vsftpd -s /sbin/nologin ftpuser

# 设置用户 ftpuser 的密码

passwd ftpuser

# 把 /home/vsftpd 的所有权给ftpuser.root

chown -R ftpuser.root /home/vsftpd

3.创建虚拟用户信息文件

vsftp目录下

cd /etc/vsftpd/

创建用户信息文件

touch vuser_passwd

vim vuser_passwd

\#编辑如下内容，创建虚拟账户信息，奇数行为用户名，偶数行为密码

ftp-user1

123456

ftp-user2

123456

4.生成虚拟用户数据文件

db_load -T -t hash -f /etc/vsftpd/vuser_passwd /etc/vsftpd/vuser_passwd.db

chmod 600 /etc/vsftpd/vuser_passwd.db

在当前文件夹下生成一个vuser_passwd.db文件

5.编辑pam认证文件

vim /etc/pam.d/vsftpd

将其他都注释掉，添加下面两行；

注：db=/etc/vsftpd/vuser_passwd 中的vuser_passwd 是你生成的虚拟用户的db文件,这里不要加扩展名。

系统为32位：

auth required pam_userdb.so db=/etc/vsftpd/vuser_passwd account

required pam_userdb.so db=/etc/vsftpd/vuser_passwd

系统为64位：

auth required /lib64/security/pam_userdb.so db=/etc/vsftpd/vuser_passwd

account required /lib64/security/pam_userdb.so db=/etc/vsftpd/vuser_passwd

查看系统位数

getconf LONG_BIT

6.为虚拟账户创建访问根目录，要在宿主用户下

# 下面是目录结构

/home/vsftpd

├──ftp-user1

│         └── files

└──ftp-user2

```
└──files
```

修改权限

chmod 777 ftp-user1

chmod 777 ftp-user2

7.创建虚拟用户配置目录

cd /etc/vsftpd/

\#创建上述配置文件中配置的虚拟用户文件夹，一定要对应

mkdir vuser_conf

cd vuser_conf

\#创建虚拟用户配置文件，文件名称要与虚拟用户名称相同

\#这里我们配置两个虚拟用户就创建两个配置文件

touch ftp-user1 ftp-user2

\#编辑两个文件，加入以下信息(注意加粗部分要替换为你虚拟账户要访问的根目录名称)

local_root=/home/vsftpd/ ftp-user1

write_enable=YES

anon_umask=022

anon_world_readable_only=NO

anon_upload_enable=YES

anon_mkdir_write_enable=YES

anon_other_write_enable=YES

8.创建chroot_list

cd /etc/vsftpd

\#创建使当前配置的虚拟用户允许访问

[root@localhost vsftpd]# touch chroot_list

[root@localhost vsftpd]# vim chroot_list

\#写入虚拟用户名

ftp-user1

ftp-user2

9.重启VSFTP服务

systemctl restart vsftpd.service

10.可以在windows中访问测试，或者使用FileZilla连接

问题汇总：

1、解决报错 500 OOPS: vsftpd: refusing to run with writable root inside chroot()

添加配置

allow_writeable_chroot=YES # 如果启用了限定用户在其主目录下需要添加这个配置。

2、登录报错530，日志显示

pam_unix(vsftpd:auth): check pass; user unknown

检查一下/etc/pam.d/vsftpd文件配置，一般是pam认证文件配置错误。