---
title: docker搭建内网穿透的工具
tags:
  - 内网穿透
categories:
  - 技术
date: 2020-03-15 18:48:08
keywords:
description:
mathjax:
katex:
hide:
---
> 想当初才用内网穿透的时候,是家里老电脑的一个http服务想外网访问,然后才搜了下,只不过当时只有什么花生壳还有什么nat什么的,不好用,而且还收费的好像.现在免费的软件多了,只不过人家花生壳有自己的产品生态,现在肯定是更好用了.当然,这是后话了.

![](https://raw.githubusercontent.com/yzddhr/yzddhr.github.io/tuchuang/img/2.png)



## 环境
- docker
- docker-compose


## 项目安装
### 项目地址

Github地址：https://github.com/khvysofq/proxyer
### 安装
1. 下载docker-compose.yml文件

```
wget https://raw.githubusercontent.com/khvysofq/proxyer/master/docker-compose.yml
```

2. 设置环境变量

```
export PROXYER_PUBLIC_HOST=1.1.1.1
```
将1.1.1.1修改成自己的IP

2. 启动

```
docker-compose up -d
```

4. 说明
- ip:6789  //访问地址
- 进去之后设置客户端认证密码

5. 其他
centos建议关闭防火墙,或者打开部分端口

```
#CentOS 6系统
service iptables stop
chkconfig iptables off

#CentOS 7系统
systemctl stop firewalld
systemctl disable firewalld
```


## 客户端
全平台支持,比较成熟。
Windows可以直接下载界面版本，然后双击可执行文件，会弹出一个网页界面，输入上面的认证密码，即可开始配置穿透。

Linux下载压缩包后，解压出二进制文件，直接在当前目录使用./proxyer命令运行即可。

最后使用起来还是很简单的，由于是新项目，功能可能不是很丰富，看作者后期会不会慢慢完善了。
