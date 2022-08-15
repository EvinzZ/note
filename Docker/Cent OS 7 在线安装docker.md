# Cent OS 7 在线安装docker

## 使用 `root` 权限登录 Centos。确保 yum 包更新到最新。

```Bash
yum update
```

## 卸载旧版本(如果安装过旧版本的话)

```Bash
yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-selinux \
                docker-engine-selinux \
                docker-engine
```

## 安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```Bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

## 设置yum源

```Bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

## 可以查看所有仓库中所有docker版本，并选择特定版本安装

```Bash
yum list docker-ce --showduplicates | sort -r
```

## 安装docker，版本号自选

```Bash
yum install docker-ce-17.12.0.ce
```

## 启动并加入开机启动

```Bash
$ sudo systemctl start docker
$ sudo systemctl status docker
$ sudo systemctl enable docker
```