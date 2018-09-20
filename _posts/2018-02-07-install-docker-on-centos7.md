---
layout: post
title: Centos7安装docker
subtitle: Centos7安装docker
tags: [Docker,Centos7]
category: Linux
---


## 安装docker 

1. 卸载老版本的 Docker 及其相关依赖

	sudo yum remove docker docker-common container-selinux docker-selinux docker-engine
	

2. 安装 yum-utils，它提供了 yum-config-manager，可用来管理yum源

	```
	sudo yum install -y yum-utils
	```
3. 添加yum源

	```
	sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
	```
4. 更新yum索引

	```
	sudo yum makecache fast
	```
5. 安装 docker-ce

	```
	sudo yum install docker-ce
	```
6. 启动 docker

	```
	sudo systemctl start docker
	```
7. 验证是否安装成功

	```
	sudo docker info
	```


## 启动 docker

	[root@localhost ~]# service docker start
	[root@localhost ~]# chkconfig docker on
	（LCTT 译注：此处采用了旧式的 sysv 语法，如采用CentOS 7中支持的新式 systemd 语法，如下：
	
	[root@localhost ~]# systemctl start docker.service
	[root@localhost ~]# systemctl enable docker.service
	


 如果还没有docker group就添加一个：

    $ sudo groupadd docker
    # 将用户加入该group内。然后退出并重新登录即可生效。
    $ sudo gpasswd -a ${USER} docker
    # 重启docker
    $ sudo service docker restart	
## 安装docker-compose
    
    curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

    sudo mkdir -p /etc/docker
    sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://xt3b6cxm.mirror.aliyuncs.com","http://hub-mirror.c.163.com"],
      "insecure-registries": ["registry.docker.td.internal"]
    }
    EOF
    sudo systemctl daemon-reload
    sudo systemctl restart docker



## 备注
 
1. docker权限问题 version 12

[详细参见](https://stackoverflow.com/questions/24288616/permission-denied-on-accessing-host-directory-in-docker)

```
up vote
156
down vote
It is an selinux issue.

You can temporarily issue

su -c "setenforce 0"
on the host to access or else add an selinux rule by running

chcon -Rt svirt_sandbox_file_t /path/to/volume

```

