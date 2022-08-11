# docker容器不能stop，rm的处理办法

`docker ps`  #找到容器的CONTAINER ID

解决办法

```
ps -aux | grep CONTAINER ID
kill -9 PID
```