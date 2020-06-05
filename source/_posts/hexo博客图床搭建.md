title: hexo博客图床搭建
date: 2020-04-08T12:53:06.432Z
tags: [技术]
categories: [技术]
---
![1.png4255553276218449162.png](https://bolo-tuchuang.oss-cn-beijing.aliyuncs.com/2020/04/08/ed76c0408082602.png)

> 作为一个hexo博客,最操蛋的就是图床了,不能直接拖拽图片上传,比起solo博客显得有点麻烦,那么就需要有一个图床来使用,现在普遍用的是本地的如那件图床或者浏览器插件之类.都不太方便,介绍自己搭建一个好用快捷的图床

## 图床推荐
推荐几个还算好用的图床
- picgo
- 新浪图床插件

## 搭建环境
安装docker环境
docker安装

```
#CentOS 6
rpm -iUvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum update -y
yum -y install docker-io
service docker start
chkconfig docker on

#CentOS 7、Debian、Ubuntu
curl -sSL https://get.docker.com/ | sh
systemctl start docker
systemctl enable docker
```

docker-compose安装

```
curl -L "https://get.daocloud.io/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## 编写Dockerfile+docker-compose.yml文件
因为本程序需要Java1.8的环境以及mysql的环境,下面用docker解决
0. 程序地址:https://github.com/Hello-hao/Tbed

### 下载最新realease版本
[下载](https://github.com/Hello-hao/Tbed/releases)

### 解压所得文件,编写Dockerfile文件

```
docker pull majiajue/jdk1.8  //先拉个镜像
```

```
FROM majiajue/jdk1.8
RUN mkdir /usr/src/myapp/
COPY . /usr/src/myapp/
WORKDIR /usr/src/myapp/
RUN chmod 777 /usr/src/myapp/
EXPOSE 8088
ENTRYPOINT  ["java","-jar","Tbed.jar"]
```
注意不要使用官方的java镜像,高版本的java不能用,只能是1.8版本,然后将解压的`Tbed.jar`文件和`application.properties`文件和`Dockerfile`放在一个文件夹下.

### 编写所用的docker-compose文件

```
version: "3"

services:
  mysql:
    container_name: mysql-asmr
    image: mysql:5.7.20
    restart: always
    volumes:
      - data:/var/lib/mysql
      - conf:/etc/mysql/conf.d
    environment:
      MYSQL_ROOT_PASSWORD: passwd
    ports:
      - "666:3306"
    networks:
      - net

  tu:
    container_name: tuchuang
    image: jdk:tu
    restart: always
    ports:
      - "8088:8088"
    networks:
      - net
      
volumes:
  data:
  conf:

networks:
  net:
```
自修改数据库密码之类的.

### 导入数据库

**复制数据库到容器**

```
docker cp picturebed.sql mysql-asmr:/
```

**进入容器操作**

```
docker exec -it mysql-asmr /bin/bash
cd /
mysql -uroot -p
输入设置的密码
CREATE DATABASE `picturebed` CHARACTER SET utf8 COLLATE utf8_general_ci;
use picturebed;
source picturebed.sql;
```
显示OK说明导入完成.

**后续操作**
可以利用`docker volume inspect data/conf`查询所在位置,然后复制文件夹到本地,然后改` - data:/var/lib/mysql`成`- ./data:/var/lib/mysql`即实现可数据本地化.

**修改application.properties文件**

```
spring.datasource.username=root
spring.datasource.password=passwd  //你的密码
spring.datasource.url=jdbc:mysql://mysql-asmr:3306/picturebed?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8
server.port=8088   //修改端口;不用修改
Expression=0 30 04 * * ?

mybatis.configuration.map-underscore-to-camel-case=true
mybatis.mapper-locations=classpath:mapper/*.xml
logging.level.cn.hellohao.dao=debug
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.thymeleaf.cache=false
multipart.maxFileSize=10240KB
multipart.maxRequestSize=10240KB
spring.thymeleaf.mode = LEGACYHTML5
spring.http.multipart.location=/data/upload_tmp
systemupdate=2020-04-07
```

### 启动

```
docker-compose up 
docker-compose up -d //后台启动
```
等待片刻,第一次初始化听费时间的.

  
