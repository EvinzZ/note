# 使用CentOS7.iso配置本地Yum源

1.将CentOS.iso文件拷贝到用户主目录中 即：～

```Bash
cp CentOS7.iso ~
```

2.创建iso文件将要挂载的目录

```Bash
mkdir -p /mnt/cdrom
```

3.挂载iso文件到刚刚创建的目录中

```Bash
mount -s loop CentOS7.iso /mnt/cdrom
```

4.创建repo文件Local.repo，然后在其中加入下面内容

```Bash
rm -rf /etc/yum.repos.d/*
vim /etc/yum.repos.d/Local.repo

[Local]
name=Local Yum
baseurl=file:///mnt/cdrom
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=1
```

5.测试

```Bash
yum clean all
yum list all
```