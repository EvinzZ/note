# docker安装Elasticsearch

1.设置max_map_count不能启动es会启动不起来

查看max_map_count的值 默认是65530

```Bash
cat /proc/sys/vm/max_map_count
```

重新设置max_map_count的值

```Bash
sysctl -w vm.max_map_count=262144
```

2.下载镜像并运行

```Bash
#拉取镜像
docker pull elasticsearch:7.7.0

#启动镜像
docker run --name elasticsearch -d -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 elasticsearch:7.7.0
```

参数说明

```Bash
--name表示镜像启动后的容器名称  

-d: 后台运行容器，并返回容器ID；

-e: 指定容器内的环境变量

-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
```

3.浏览器访问ip:9200

4.安装ik分词器

这里采用离线安装

下载分词器压缩包

下载地址：

https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.7.0/elasticsearch-analysis-ik-7.7.0.zip

将IK分词器上传到/tmp目录中（xftp）

将分词器安装进容器中

\#将压缩包移动到容器中

```Bash
docker cp /tmp/elasticsearch-analysis-ik-7.7.0.zip elasticsearch:/usr/share/elasticsearch/plugins
#进入容器
docker exec -it elasticsearch /bin/bash  

#创建目录
mkdir /usr/share/elasticsearch/plugins/ik

#将文件压缩包移动到ik中
mv /usr/share/elasticsearch/plugins/elasticsearch-analysis-ik-7.7.0.zip /usr/share/elasticsearch/plugins/ik

#进入目录
cd /usr/share/elasticsearch/plugins/ik

#解压
unzip elasticsearch-analysis-ik-7.7.0.zip

#删除压缩包
rm -rf elasticsearch-analysis-ik-7.7.0.zip
```

退出并重启镜像