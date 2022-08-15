# docker安装kibana

## 下载镜像

```Bash
docker pull kibana:7.6.2
```

## 配置文件

```Bash
mkdir -p /data/elk7/kibana/config/
vi /data/elk7/kibana/config/kibana.yml
```

内容如下：

```YAML
#
# ** THIS IS AN AUTO-GENERATED FILE **
#
# Default Kibana configuration for docker target
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://192.168.31.190:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true
```

## 启动

```Bash
docker run -d \
 --name=kibana \
 --restart=always \
 -p 5601:5601 \
 -v /data/elk7/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml \
kibana:7.6.2
```

## 查看日志

```Bash
docker logs -f kibana
```

## 访问页面

http://host:5601