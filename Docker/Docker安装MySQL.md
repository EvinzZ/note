# Docker安装MySQL

拉镜像

```Bash
docker pull mysql:5.7
```

运行容器

```Bash
 docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

需要指定时区才能存储的时间是对的