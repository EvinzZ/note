# Docker搭建ZK集群

## 1. 搭建集群

1. 创建网络

```bash
docker network create docker_net
```



2. 编写docker-compose.yml

```yml
version: '3.7'

networks:
  docker_net:
    external: true


services:
  zoo1:
    image: zookeeper
    restart: unless-stopped
    hostname: zoo1
    container_name: zoo1
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
    volumes:
      - ./zookeeper/zoo1/data:/data
      - ./zookeeper/zoo1/datalog:/datalog
    networks:
      - docker_net

  zoo2:
    image: zookeeper
    restart: unless-stopped
    hostname: zoo2
    container_name: zoo2
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181
    volumes:
      - ./zookeeper/zoo2/data:/data
      - ./zookeeper/zoo2/datalog:/datalog
    networks:
      - docker_net

  zoo3:
    image: zookeeper
    restart: unless-stopped
    hostname: zoo3
    container_name: zoo3
    ports:
      - 2184:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
    volumes:
      - ./zookeeper/zoo3/data:/data
      - ./zookeeper/zoo3/datalog:/datalog
    networks:
      - docker_net
```



3. 启动集群

```bash
docker-compose -f docker-compose.yml up -d
```



4. 测试集群

- 查看zoo1角色

```bash
➜  docker docker exec -it zoo1 /bin/sh
# zkServer.sh status         
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

- 查看zoo2角色

```bash
➜  docker docker exec -it zoo2 /bin/sh
# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

- 查看zoo3角色

```bash
➜  docker docker exec -it zoo3 /bin/sh
# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader
```

- 查看zoo3选举数据

```bash
➜  docker echo srvr | nc localhost 2184
Zookeeper version: 3.5.6-c11b7e26bc554b8523dc929761dd28808913f091, built on 10/08/2019 20:18 GMT
Latency min/avg/max: 0/0/0
Received: 2
Sent: 1
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: leader
Node count: 5
Proposal sizes last/min/max: -1/-1/-1
```



## 2.选举演练

### 2.1 模拟Leader掉线

```bash
➜  zoo1 docker stop zoo3
zoo3
```

查看此时的选举结果(操作同查看角色操作步骤)。可以看到Zookeeper集群重新选举结果: zoo2被选为leader

### 2.2 **zoo3节点重新上线**

```bash
➜  zoo1 docker start zoo3
zoo3
```

查看zoo3角色，发现zoo3自动作为follower加入集群。

*注意：查看currentEpoch中的数值，存储值为2，代表经过了2次选举。第一次为刚启动时触发选举，第二次为leader宕机后重新选举*

## 3. 常用操作

### 3.1 查看文件目录

笔者本地有安装的Zookeeper环境，所以这里用本地的zkCli进行测试。

```bash
zkCli -server localhost:2182,localhost:2183,localhost:2184
Connecting to localhost:2182,localhost:2183,localhost:2184
Welcome to ZooKeeper!
JLine support is enabled

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 0] ls /
[zookeeper]
```

### 3.2 创建顺序节点

顺序节点保证znode路径将是唯一的。

```bash
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 1] create -s /zk-test 123
Created /zk-test0000000000
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 2] ls /
[zk-test0000000000, zookeeper]
```

### 3.3 创建临时节点

当会话过期或客户端断开连接时，临时节点将被自动删除

```bash
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 3] create -e /zk-temp 123
Created /zk-temp
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 4] ls /
[zk-test0000000000, zookeeper, zk-temp]
```

临时节点在客户端会话结束后就会自动删除，下面使用quit命令行退出客户端,再次连接后即可验证。

```bash
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 5] quit
Quitting...
➜  docker zkCli -server localhost:2182,localhost:2183,localhost:2184
Connecting to localhost:2182,localhost:2183,localhost:2184
Welcome to ZooKeeper!
JLine support is enabled

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 0] ls /
[zk-test0000000000, zookeeper]
```

### 3.4 创建永久节点

```bash
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 1] create /zk-permanent 123
Created /zk-permanent
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 2] ls /
[zk-permanent, zk-test0000000000, zookeeper]
```

### 3.5 读取节点

```bash
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 3] get /

cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x400000008
cversion = 3
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 3
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 4] ls2 /
[zk-permanent, zk-test0000000000, zookeeper]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x400000008
cversion = 3
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 3
```

使用 ls2 命令来查看某个目录包含的所有文件，与ls不同的是它查看到time、version等信息

### 3.6 更新节点

```bash
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 5] set /zk-permanent 456
cZxid = 0x400000008
ctime = Tue Mar 03 21:35:20 CST 2020
mZxid = 0x400000009
mtime = Tue Mar 03 21:40:11 CST 2020
pZxid = 0x400000008
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```

### 3.7 检查状态

```bash
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 6] stat /zk-permanent
cZxid = 0x400000008
ctime = Tue Mar 03 21:35:20 CST 2020
mZxid = 0x400000009
mtime = Tue Mar 03 21:40:11 CST 2020
pZxid = 0x400000008
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```

### 3.8 删除节点

```bash
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 7] rmr /zk-permanent
[zk: localhost:2182,localhost:2183,localhost:2184(CONNECTED) 8] ls /
[zk-test0000000000, zookeeper]
```