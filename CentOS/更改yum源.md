# 更改yum源

1、打开mirrors.aliyun.com，选择centos的系统，点击帮助

2、执行命令：`yum install wget -y`

3、改变某些文件的名称

​               ` mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup        `      

4、执行更换yum源的命令

CentOS 6:

​                `wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo    `          

CentOS 7:

​                `wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo              `

CentOS 8:

​                `wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo              `

5、更新本地缓存

​                `yum clean all yum makecache `