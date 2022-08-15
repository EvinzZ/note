# CentOS Java程序自启

# 脚本

设置开机脚本和关闭脚本

[wbs-service-start.sh](http://wbs-service-start.sh)

```Bash
#!/bin/sh

export JAVA_HOME=/usr/java/jdk
export PATH=$JAVA_HOME/bin:$PATH

nohup java -jar /usr/java/wbs-service.jar > /dev/null  2>&1 &
echo $! > /var/run/wbs-service.pid
```

[关闭脚本wbs-service-stop.sh](http://xn--wbs-service-stop-qu10a737zq70ejbyd.sh)

```Bash
#!/bin/sh
PID=$(cat /var/run/wbs-service.pid)
kill -9 $PID
```

chmod +x xxx.sh文件，对于启动和关闭的sh文件一定要改权限，否则启动服务的时候会报错的。

# 设置服务

cd /usr/lib/systemd/system

vim wbs-service.service（可以修改你自己要起得名称）

```Bash
#!/bin/sh

[Unit]
Description=wbs-service
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/wbs-service-start.sh
ExecStop=/usr/wbs-service-stop.sh
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

到最后别忘了 systemctl daemon-reload 重新加载一下， 再运行

systemctl start test.service

这样你就可以像操作其他的服务那样，使用systemctl来操作你自己的java程序了，比如要把这个服务加入到开机启动中：

systemctl enable test.service