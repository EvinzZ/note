# Docker安装kibana
```bash
docker pull kibana:7.7.0
mkdir - p /root/kibana/config/
vim /root/kibana/config/kibana.yml


#
# ** THIS IS AN AUTO-GENERATED FILE **
#

# Default Kibana configuration for docker target
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://192.168.31.190:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true


docker run -d   --name=kibana   --restart=always   -p 5601:5601   -v /root/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml   kibana:7.7.0
```