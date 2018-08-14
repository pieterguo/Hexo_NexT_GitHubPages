---
title: 从零开始建立 OTA 服务和 MQTT 服务
tags: OTA 
---

## 背景

- 为补齐在线自动升级这块短板，让开发成果和用户体验之间形成闭环。
- 家长控制 APP 功能

## 阿里云

![aliyun_logo](http://note.youdao.com/yws/res/15128/60C6EF1955E240399357E327CA54D64F)

阿里云计算 , Alibaba Cloud （全称阿里云计算有限公司，简称阿里云），是一家提供云端运算服务的科技公司，创立于2009年9月，为阿里巴巴集团全资所有。[1]阿里云计算公司总部位于杭州，在北京和硅谷设有机构，研发和运营涉及云计算的产品与服务[2]。竞争对手：亚马逊AWS、腾讯云。

- 云服务器 ECS（Elastic Compute Service）是一种弹性可伸缩的计算服务
- ~~MQS 消息服务~~

## CentOS

![CentOS logo](https://upload.wikimedia.org/wikipedia/commons/b/bf/Centos-logo-light.svg)

CentOS（Community Enterprise Operating System）是Linux发行版之一，它是来自于Red Hat Enterprise Linux依照开放源代码规定发布的源代码所编译而成。两者的不同，在于CentOS并不包含封闭源代码软件。CentOS 对上游代码的主要修改是为了移除不能自由使用的商标。


## Spring
![Spring logo](https://i.pinimg.com/originals/2a/4d/4b/2a4d4bc85a5100cf62fcdbaaabbb38d0.png)



Spring Framework 是一个开源的Java／Java EE全功能栈（full-stack）的应用程序框架，以Apache License 2.0开源许可协议的形式发布





## start.spring.io
![image](http://note.youdao.com/yws/res/15173/FB1B3B565BA446A2A2251CD54FC4360C)



## IntelliJ IDEA
![image](https://linuxhint.com/wp-content/uploads/2016/05/IntelliJ-IDEA-img1.jpg)



## MQTT
![image](https://www.eclipse.org/paho/images/mqttorg-glow.png)
- 什么是 MQTT

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输）是IBM开发的一个即时通讯协议，有可能成为物联网的重要组成部分。该协议支持所有平台，几乎可以把所有联网物品和外部连接起来，被用来当做传感器和制动器（比如通过Twitter让房屋联网）的通信协议。

[ MQTT 3.1.1 协议规范](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718029)
http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718029



## Eclipse Mosquitto™

Eclipse Mosquitto is an open source (EPL/EDL licensed) message broker that implements the MQTT protocol versions 3.1 and 3.1.1. Mosquitto is lightweight and is suitable for use on all devices from low power single board computers to full servers.
- mosquitto-1.4.11.tar.gz 
- - BUG： Retaind MSG 不能保存在 MQTT SERVER

- mosquitto-1.4.15.tar.gz 
- - Update Last Will MSG 进行异常掉线检测


## Paho Client
![image](https://www.eclipse.org/paho/images/paho_logo_400.png)



## Docker

![image](https://upload.wikimedia.org/wikipedia/commons/7/79/Docker_%28container_engine%29_logo.png)

Docker是一个开放源代码软件项目，让应用程序布署在软件容器下的工作可以自动化进行，借此在Linux操作系统上，提供一个额外的软件抽象层，以及操作系统层虚拟化的自动管理机制[1]。Docker利用Linux核心中的资源分脱机制，例如cgroups，以及Linux核心名字空间（name space），来创建独立的软件容器（containers）。这可以在单一Linux实体下运作，避免引导一个虚拟机造成的额外负担[2]。

- MYSQLD
- OTAD SERVER
- MQTTD SERVER

## MYSQLD

```
$ docker run --name mysqld -e MYSQL_ROOT_PASSWORD=Password@1 -v /home/admin/demo/mysql:/var/lib/mysql -p 3306:3306 -d mysql/mysql-server:5.7 --character-set-server=utf8 --collation-server=utf8_general_ci 
```
## OTA SERVER

Build otad
```
docker build -t smartcast/otaserver .
docker stop otad
docker rm otad
docker run -tid --name otad -p 8080:8080  -v /home/versions:/home/versions smartcast/otaserver

```

Dockfile

```
FROM ibmjava:jre
MAINTAINER pieter

COPY offical-smartcast-OTAImg.jar /opt/ota.jar 
 
ENTRYPOINT ["java", "-jar" , "/opt/ota.jar"]

```



## MQTT SERVER

```
docker build -t smartcast/mqttserver .
docker stop mqttd
docker rm mqttd
docker run -tid --name mqttd -p 1883:1883 -p 8883:8883 smartcast/mqttserver

```


```
FROM ubuntu

MAINTAINER PieterGuo@gmail.com

RUN apt-get update -y

# Install mosquitto
RUN apt-get install -y cmake g++ python uuid-dev libc-ares-dev libwrap0-dev xsltproc docbook-xsl
RUN mkdir -p /build/mosquitto
COPY Source /build/mosquitto
WORKDIR /build/mosquitto
RUN touch mosquitto.log
RUN ls
RUN ls ../
RUN make
RUN make install
RUN useradd -r -m -d /var/lib/mosquitto -s /usr/sbin/nologin -g nogroup mosquitto
RUN cp /etc/mosquitto/mosquitto.conf.example /etc/mosquitto/mosquitto.conf

# Everything ready
VOLUME ["/etc/mosquitto", "/var/lib/mosquitto"]
WORKDIR /etc/mosquitto
#EXPOSE 1883 8883 8884 8080 8081

CMD ["/usr/local/sbin/mosquitto", "-c", "/etc/mosquitto/mosquitto.conf"]

```

## Enjoy Docker

```
$ docker start mysqld
$ docker start otad
$ docker start mqttd

```





## 运营问题

- DDOS

SSH登录安全策略检测如下配置：
 1. 登录端口是否为默认22端口
 2. root账号是否允许直接登录
 3. 是否使用不安全的SSH V1协议
 4. 是否使用不安全的rsh协议
 5. 是否运行基于主机身份验证的登录

修复方案：
编辑 
/etc/ssh/sshd_config
```
1.Port（非22）
2.PermitRootLogin（no）
3.Protocol（2）
4.IgnoreRhosts（yes）
5.HostbasedAuthentication（no）
```
