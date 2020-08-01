---
title: 搭建品葱论坛-docker
urlname: efoa4d
date: 2020-06-09 22:02:43 +0800
tags: []
categories: []
---

## 简介

- 品葱论坛本来是仿照知乎搭建的论坛，现在已沦为（颜色）蛆乱骂之地
- 还是挺方便的自己搭建一个自己用
- PS : 语雀会员的领取 [https://51.ruyo.net/15979.html](https://51.ruyo.net/15979.html)

## 环境

- 安装 docker
- 安装 docker-compose

```shell
#安装Docker

curl -sSL https://get.docker.com/ | sh
service docker start

#安装Docker Compose
curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## 指令

- 因为是 docker 本来就是一件命令的事情

```shell
docker run -d \
  --name pincong-wecenter \
  -p 80:80 \
  -v $PWD/sql:/var/lib/mysql \
  -v $PWD/logs:/data/logs \
  -v $PWD/www:/data/htdocs \
  xmader/pincong-wecenter
```

- 导入数据库
  1.  `sudo docker exec -it pincong-wecenter bash`
  1.  mysql -h 127.0.0.1 -P 3306
  1.  show databases;
  1.  use db
  1.  source /data/htdocs/install/db/tables.sql
  1.  source /data/htdocs/install/db/settings.sql

* 其他说明

初始的密码和用户名都是 admin
