---
title: Docker安装&常用指令
date: 2018-10-18 15:56:05
categories:
- docker
---

学习docker容器基本指令，以及尝试在docker中使用redis数据库。

<!--more-->

## 1.安装后启动报错

安装docker完毕后启动报错，提示“Hardware assisted vitrualization and data execution protection must be enabled in the BIOS”

这个错误大概率是由于磁盘没有开启虚拟化功能导致的。解决：

1、开机时按F2进入BIOS（不同机器可能会略有区别）

2、进入Security --> Virtualization

3、调整Intel VT-Virtualization-Technology：**Enabled**

## 2.Docker常用命令

### 2.1 docker镜像操作命令 

1. 查看docker版本信息

```shell
docker -v
```

2. docker镜像搜索（或在docker hub网页搜索）

```shell
docker search 'image-name'
```

3. 镜像下载

```shell
docker pull 'image-name'
```

4. 镜像删除

```shell
docker rmi 'image-name'
```

5. 删除所有镜像

```shell
docker rmi $(docker images -q)
```



### 2.2 docker容器命令

1. 启动一个镜像为容器

```shell
docker run --name 'container-name' -d 'image-name'
docker run --help //查看run命令参数
```

2. 查看docker中容器列表

   ```shell
   docker ps	//查看运行中的容器
   docker ps -a //查看运行、停止状态的容器
   ```

3. 停止&启动容器

```shell
docker stop 'container-name/id'
docker start 'container-name/id'
```

4. 容器端口映射

```shell
//启动一个叫port-redis的容器，映射容器6379端口到本机6378
docker run -d -p 6378:6379 --name port-redis redis
```

> windows下运行的docker实际上是在虚拟机中，上述的端口映射是docker-->虚拟机，如果想在本地开发环境使用，还需要一次映射：虚拟机-->开发机

5. 删除容器

```shell
docker rm 'container-name/id'
docker rm $(docker ps -a -q)	//删除所有容器
```

6. 容器日志

```shell
docker logs 'container-name/id'
```

7. 登录容器

```shell
docker exec -it 'container-name/id' bash
```

> 运行中的容器实际上相当于一个完备的Linux系统，可以登录后进行常规Linux命令操作

8. 登录后查看容器信息

按上述登录到容器中后就可以按Linux的命令如hostname、env、ip addr等进行查看容器的信息。补充另外另种方式：

* 不用进入容器，直接通过命令查看

```shell
docker exec 'container-name/id' hostname
docker exec 'container-name/id' env
```

* 使用**inspect**命令，可以获得全部内容

```shell
docker inspect 'container-name/id'
docker inspect -f {{.NetworkSettings.IPAddress}} 'container-name/id'
docker inspect 'container-name/id'| grep IPAddress
```

> -f：format指令可以设定具体的属性内容 :slightly_smiling_face:
>
> inspect命令结合grep使用快速查询容器属性！

## 3. 虚拟机端口映射配置

上面介绍了docker容器的端口映射，在windows下还要进行虚拟机到一般开发环境的端口映射才可以正常访问容器。以redis为例说明：

### 3.1虚拟机端口映射到容器端口

1. 开启容器并映射**虚拟机port：容器port**

```shell
docker run -d -p 6379:6379 --name test-redis redis
-p：小写表示指示具体的端口映射
-P：大写表示随机分配主机端口映射到容器开放的网络端口上。
```

将虚拟机的6379端口映射到容器6379端口

2. 映射时指定ip地址

一般不需要特别指定虚拟机ip地址，默认0.0.0.0，也可以在绑定时指定：

```shell
docker run -d -p 127.0.0.1:6379:6379 --name test-redis redis
#启动结果如下
CONTAINER ID        IMAGE               COMMAND                  CREATED             
2aedbac0a984        redis               "docker-entrypoint.s…"   4 seconds ago       

STATUS              PORTS                      NAMES	
Up 3 seconds        127.0.0.1:6379->6379/tcp   test-redis
```

3. 映射时可以指定通信协议tcp、udp

```shell
docker run -d -p 127.0.0.1:6379:6379/tcp --name test-redis redis
```

4. 查看容器端口绑定&IP地址

```shell
docker port test-redis
#结果：容器端口映射到虚拟机127.0.0.1:6379
6379/tcp -> 127.0.0.1:6379

docker inspect test-redis|grep 'IPAddress'
#结果
"SecondaryIPAddresses": null,
"IPAddress": "172.17.0.2",
"IPAddress": "172.17.0.2",
```

> 如果想要同时绑定多个端口，则使用多个-p指令即可
>
> 除了启动容器时绑定port，也可以通过宿主机的iptables进行nat转发实现，详见：[docker-redis](https://blog.csdn.net/mnasd/article/details/83087881)

### 3.2映射本机端口到虚拟机端口

如果想要在本机上使用，还需要进行本机端口-->虚拟机端口映射的操作。

```shell
#1.查看目前端口的映射情况
netsh interface portproxy show v4tov4
netsh interface portproxy show v4tov4 | find "127.0.0.1"

#2.增加一个端口映射,listen主机IP&port --> connect 虚拟机IP&port
netsh interface portproxy add v4tov4 listenaddress=127.0.0.1 listenport=6379 connectaddress=127.0.0.1 connectport=6379

#3.删除一个端口映射
netsh interface portproxy delete v4tov4 listenaddress=/address listenport=/port
```

## 4. Docker中使用redis

经过以上的操作我们直接通过127.0.0.1:6379即可连接redis。我们也可以直接登录容器操作redis数据库。

```shell
#1.进入docker的redis-cli环境
docker exec -it 'container-name/id' redis-cli
#例如：
docker exec -it test-redis redis-cli
#进入如下界面即可使用redis命令，如：keys、get、ttl等，退出使用exit即可
127.0.0.1:6379>
```

