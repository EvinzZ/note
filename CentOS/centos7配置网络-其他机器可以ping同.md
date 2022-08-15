# centos7配置网络-其他机器可以PING通

1. 虚拟机配置-网络适配器-桥接模式

![1660532364059](../images/1660532364059.jpg)

2. 查看本机的网卡信息

![e1d89355215356868230d3c917f7046](../images/e1d89355215356868230d3c917f7046.jpg)

3. 修改`/etc/sysconfig/network-scripts/ifcfg-ens33`

![image-20220815110901414](../images/image-20220815110901414.png)

4. 重启网卡服务`systemctl restart network`
5. 测试`ping www.baidu.com`
6. 测试`ping 本机ip`