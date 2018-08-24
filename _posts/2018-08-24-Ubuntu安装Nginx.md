---
layout:     post
title:      Ubuntu安装编译Nginx
subtitle:	  Ubuntu Nginx
date:       2018-08-24
author:     Neil
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - Ubuntu
    - Nginx
---

# Ubuntu版本
Mac上使用VirtualBox安装的Ubuntu系统版本
![系统版本](https://ws4.sinaimg.cn/large/006tNbRwly1fukxqqr3z0j31kw04twfk.jpg)

# 下载源码
```
wget http://nginx.org/download/nginx-1.0.10.tar.gz
```
# 解压
```
tar -zxvf nginx-1.0.10.tar.gz
```

# 环境检查
```
./configure 
```
如果出现error : the HTTP rewrite module requires the PCRE library 需要安装PCRE库 OpenSSL库

```
sudo apt-get install libpcre3 libpcre3-dev  
sudo apt-get install openssl libssl-dev 
```


# 编译安装
```
cd nginx-1.0.10
./configure 
make
make install
```

# 启动
```
sudo /usr/local/nginx/sbin/nginx
```

![系统版本](https://ws1.sinaimg.cn/large/006tNbRwly1fukyqhvg1bj31kw06m0tu.jpg)

备注其他操作 比如停止nginx

```
sudo /usr/local/nginx/sbin/nginx -t #检测配置文件是否正确 
sudo /usr/local/nginx/sbin/nginx -s stop #停止 
sudo /usr/local/nginx/sbin/nginx -s reload #重载配置文件 
```


# 参考：  
[实战开发一个Nginx扩展 (Nginx Module)](https://segmentfault.com/a/1190000009769143)  
[在Ubuntu上编译安装Nginx (Nginx Module)](https://vsxen.github.io/2017/04/09/nginx-sourse-compile-on-ubuntu/)

 









 

