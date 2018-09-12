---
title: Nginx实现Tomcat隐藏端口号
date: 2018-02-05
author: asing1elife
categories:
 - nginx
 - tomcat
tags:
 - nginx
 - tomcat
---
> Tomcat默认可以使用80端口实现隐藏端口号，但80端口权限过高，通常会被防火墙禁用无法直接访问  
> 但是可以通过Nginx反向代理实现监听80端口转发到对应的Tomcat服务器  

## 实现方式
1. 在Nginx的配置文件 `nginx.conf` 中添加如下配置
	* 该文件的在 `/usr/local/etc/nginx/` 下

```sh
server {
	listen       80;
	server_name  127.0.0.1;
	server_name_in_redirect off;
	proxy_set_header Host $host:$server_port;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header REMOTE-HOST $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	location / {
		proxy_pass http://127.0.0.1:8090/;
	}
}
```

2. 分别启动Tomcat和Nginx即可实现隐藏端口号直接访问的效果