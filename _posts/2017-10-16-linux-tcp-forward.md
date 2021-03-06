---
layout: post
title: Linux配置端口转发
subtitle: Linux配置端口转发
tags: [Linux, 端口转发]
category: Linux
---

` ssh -C -f -N -g root@192.168.0.1 -L 20022:192.168.0.2:20022`

应用场景举例：
实现下面两个Shell

```
$ mysql -h10.13.34.119 -P3306 -uroot -p123456
$ mysql -h10.13.34.120 -P3307 -uroot -p123456
```
都是同样访问 10.13.34.119:3306 这个数据库服务器
这时候我们就需要将 10.13.34.120:3307 映射到 10.13.34.119:3306

##第一种：利用iptable

iptables，大家都知道，网络防火墙，就是用于实现Linux下访问控制的东东
先打开IP转发：

echo 1 > /proc/sys/net/ipv4/ip_forward
添加NAT规则，并重启iptables：

```
service iptables stop
iptables -t nat -A PREROUTING --dst 10.13.34.119 -p tcp --dport 3306 -j DNAT --to-destination 10.13.34.120:3307
iptables -t nat -A POSTROUTING --dst 10.13.34.120 -p tcp --dport 3307 -j SNAT --to-source 10.13.34.119
service iptables save
service iptables start

```

##第二种：使用SSH协议转发

ssh的三个强大的端口转发命令：

转发到远端：ssh -C -f -N -g -L 本地端口:目标IP:目标端口 用户名@目标IP
转发到本地：ssh -C -f -N -g –R 本地端口:目标IP:目标端口 用户名@目标IP
ssh -C -f -N -g -D listen_port user@Tunnel_Host
这里我们使用第一种，只需在 10.13.34.120 这个机器执行以下命令：

```$ ssh -C -f -N -g root@10.121.34.119 -L 3307:10.121.34.119:3306```
-C ：压缩数据传输。
-f ：后台认证用户/密码，通常和 -N 连用，不用登录到远程主机。
-N ：不执行脚本或命令，通常与 -f 连用。
-g ：在-L/-R/-D参数中，允许远程主机连接到建立的转发的端口，如果不加这个参数，只允许本地主机建立连接。
-L ：转发规则，本地端口:目标IP:目标端口

如此，当你使用

mysql -h10.13.34.120 -P3307 -uroot -p123456
时，就会自动转发到 10.121.34.119:3306


