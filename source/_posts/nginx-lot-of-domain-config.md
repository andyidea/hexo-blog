---
title: nginx多域名配置
date: 2017-02-11 14:40:10
tags: [nginx]
---

nginx多域名配置是在配置文件中建立多个`server`配置，在每个`server`配置中用`server_name`来对域名信息进行过滤。
举个例子，下面是一个`conf`文件：
	
```nginx
server 
{ 
listen  80; 
server_name www.web1.com;             #绑定域名 
index index.htm index.html index.php;  #默认文件 
root /home/www.web1.com;              #网站根目录
include location.conf;                 #调用其他规则，也可去除
}

server 
{ 
listen  80; 
server_name www.web2.com;             #绑定域名 
index index.htm index.html index.php;  #默认文件 
root /home/www/web2.com;              #网站根目录
include location.conf;                 #调用其他规则，也可去除
}
```

以上配置信息就是在一个nginx配置中最简单的多域名配置方法，关于`server_name`，nginx官方还提供了很多正则匹配的过滤方式，详情请看[nginx官方文档](http://nginx.org/en/docs/http/server_names.html)。

##注意事项
特别要注意的是，在nginx的配置文件中只有一个`server`配置的时候，`server_name`是`无效`的，也就是说任何域名绑定了这个IP的时候，无论`server_name`填什么域名，都会匹配到这个唯一的`server`。只有在多个`server`的时候，`server_name`才会有效。