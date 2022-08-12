# Tomcat安装

1.安装wget

```bash
yum install -y wget
```

2.确认是否安装成功

```bash
rpm -qa | grep wget
```

3.下载安装包

```bash
wget http://mirrors.shu.edu.cn/apache/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.tar.gz
```

4.解压

```bash
tar -zxvf apache-tomcat-8.5.37.tar.gz
```

5.启动

```bash
./startup.sh

// 下面为响应
Using CATALINA_BASE: /root/apache-tomcat-8.5.37
Using CATALINA_HOME: /root/apache-tomcat-8.5.37
Using CATALINA_TMPDIR: /root/apache-tomcat-8.5.37/temp
Using JRE_HOME: /usr
Using CLASSPATH: /root/apache-tomcat-8.5.37/bin/bootstrap.jar:/root/apache-tomcat-8.5.37/bin/tomcat-juli.jar
Tomcat started.
```

6.如果外部不能访问

```bash
systemctl stop firewalld    #关闭防火墙
systemctl status firewalld  #查看状态
systemctl disable firewalld    #开机自动关闭
netstat -ntlp
```

7.Tomcat里修改JAVA_HOME和JRE_HOME

```shell
tomcat的bin目录下面的setclasspath.sh，添加如下内容，路径自己修改 

#!/bin/sh 
# ----------------------------------------------------------------------------- 
#  Set CLASSPATH and Java options 
# 
#  $Id: setclasspath.sh 467182 2006-10-23 23:47:06Z markt $ 
# ----------------------------------------------------------------------------- 

export JAVA_HOME=/usr/lib/jvm/java-8-sun 
export JRE_HOME=/usr/lib/jvm/java-8-sun/jre 

# First clear out the user classpath 
CLASSPATH= 
```

8.设置tomcat默认访问路径 /

```xml
tomcat/conf/server.xml


<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
          <Context docBase="hvi" path="/" reloadable="true" privileged="true" />     
            -------   上面这句是要修改的内容------
      </Host>
    </Engine>
```

