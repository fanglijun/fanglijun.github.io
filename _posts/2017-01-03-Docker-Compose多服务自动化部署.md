---
layout: post
title:  "Docker Compose多服务自动化部署"
date:   2017-01-03 23:39:58 +0800
categories: my2829
---

### 一、整体结构
![](https://mmbiz.qlogo.cn/mmbiz_jpg/SHQtibmBWibdzzgNR2WwkMPIf1vxoawmWTyVmukgSjfmiaSy8lfCKa3vqCibiakhic7VvVyibnv8ZA4ic29yicT45MT4gJA/0?wx_fmt=jpeg)

![](https://mmbiz.qlogo.cn/mmbiz_png/SHQtibmBWibdzzgNR2WwkMPIf1vxoawmWTfAOfApGWgzHQglqlU7YKFpC7fFfLDOmZmspfO2mPzK5HQQlYYWWhkQ/0?wx_fmt=png)

### 二、docker-compose.yml 定义服务
```yml
version: '2'
services:
    front:
        build: ./front
        ports:
            - "80:80"
    photoeditor:
        build: ./photoeditor
    myip:
        build: ./myip
```
### 三、Front Container运行nginx做出口
```nginx
# nginx典型的反向代理配置
server {
  listen       80;
  server_name  ps.jianghu.im;

  location / {
    proxy_pass http://photoeditor;
  }
}

server {
  listen       80;
  server_name  myip.jianghu.im;

  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass       http://myip;
  }
}
```

### 四、一键部署脚本 start.sh
```bash
#!/bin/sh

git clone git@bitbucket.org:username/photoeditor.git

git clone git@bitbucket.org:username/myip.jianghu.im.git myip

docker-compose up --build
```
