---
layout: post
title: Nginx做4层负载均衡
subtitle: Nginx做4层负载均衡
tags: [Nginx]
category: Nginx
---

## nginx配置TCP转发

这个模块可以实现基于TCP、UDP和Unix域的socket的协议的代理服务。这个 模块是在nginx-1.9 以后版本才添加的模块,如果要使用这个模块的话，要重新编译这个源代码，参考之前的的博客[nginx安装](http://blog.csdn.net/youbingchen/article/details/51605181)，添加编译选项`--with-stream`。就可以使用 这个模块


```
stream {

  upstream mysql_tcp_proxy {
    server 192.168.15.150:3306;
  }

  server {
    listen 3306;
    proxy_connect_timeout 1s;
    proxy_timeout 10s;  #后端连接超时时间
    proxy_pass mysql_tcp_proxy;
  }
}

```