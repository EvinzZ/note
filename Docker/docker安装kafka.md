# docker安装kafka

1:[kafka](https://so.csdn.net/so/search?q=kafka&spm=1001.2101.3001.7020)需要zookeeper管理，所以需要先安装zookeeper。 

```shell
#拉取镜像
docker pull wurstmeister/zookeeper
 
#运行容器
docker run -d --name zookeeper -p 2181:2181 \
-v /etc/localtime:/etc/localtime wurstmeister/zookeeper
```

arm64架构的服务器安装方式如下：

```shell
#拉取镜像
docker pull arm64v8/zookeeper
 
#运行容器
docker run -d --name zookeeper -p 2181:2181 \
-v /etc/localtime:/etc/localtime  arm64v8/zookeeper
```

2：安装kafka。

```shell
#拉取镜像
docker pull wurstmeister/kafka
 
#运行容器
docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=192.168.200.10:2181/kafka \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.200.10:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
-v /etc/localtime:/etc/localtime wurstmeister/kafka
```

说明：

-e KAFKA_BROKER_ID=0  在kafka集群中，每个kafka都有一个BROKER_ID来区分自己

-e KAFKA_ZOOKEEPER_CONNECT=192.168.0.2:2181/kafka 配置zookeeper管理kafka的路径

-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.0.2:9092  把kafka的地址端口注册给zookeeper

-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 配置kafka的监听端口

-v /etc/localtime:/etc/localtime 容器时间同步虚拟机的时间

3：验证kafka是否可以使用

进入kakfa容器

```shell
docker exec -it kafka /bin/sh
```

创建topic

```bash
kafka-topics.sh --create --bootstrap-server localhost:9092 \
--replication-factor 1 --partitions 1 --topic mytest
```

如下图所示，创建topic成功。

![img](image/4e2b3708e01f4edb8925e3cf433c48de.png)

查看topic

```bash
kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic mytest
```

如下图所示，可以查看到mytest topic的基本信息。

![img](image/371cb0ce1c034b8880cf4ef6ea067fa1.png)