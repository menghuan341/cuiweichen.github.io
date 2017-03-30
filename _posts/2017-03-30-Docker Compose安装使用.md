---
layout: post
title: Docker Compose安装使用文档
date: 2016-08-07
categories: blog
tags: [docker]
description: Compose

---

Docker Compose安装使用
![TEST](/assets/image/compose.png)

1.	Compose概述

Docker Compose是一个用来定义和运行复杂应用的Docker工具。使用Compose，你可以在一个文件中定义一个多容器应用，然后使用一条命令来启动你的应用，完成一切准备工作。
2.	Compose特性

一个使用Docker容器的应用，通常由多个容器组成。使用Docker Compose，不再需要使用shell脚本来启动容器。在配置文件中，所有的容器通过services来定义，然后使用docker-compose脚本来启动，停止和重启应用，和应用中的服务以及所有依赖服务的容器。完整的命令列表如下：

build 构建或重建服务
help 命令帮助
kill 杀掉容器
logs 显示容器的输出内容
port 打印绑定的开放端口
ps 显示容器
pull 拉取服务镜像
restart 重启服务
rm 删除停止的容器
run 运行一个一次性命令
scale 设置服务的容器数目
start 开启服务
stop 停止服务
up 创建并启动容器

参考： https://docs.docker.com/compose/install/

3.	Install Compose

Docker Compose可以运行在macOS、Windows和64位Linux上。在安装之前需要先安装Docker。下面将以CentOS系统为例，介绍Compose的安装步骤:

3.1系统要求

64-bit version of CentOS 7

3.2 Docker安装

此处省略

3.3 Compose安装

3.3.1 下载docker-compose

# curl -L "https://github.com/docker/compose/releases/download/1.11.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

3.3.2 添加可执行权限

# chmod +x /usr/local/bin/docker-compose

3.3.3 安装command completion

A. bash
# curl -L https://raw.githubusercontent.com/docker/compose/1.11.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
B. Zsh
# mkdir -p ~/.zsh/completion 
# curl -L https://raw.githubusercontent.com/docker/compose/1.11.2/contrib/completion/zsh/_docker-compose > ~/.zsh/completion/_docker-compose

在~/.zshrc中添加：

fpath=(~/.zsh/completion $fpath)
autoload -Uz compinit && compinit –i

3.3.4 Test the installation

# docker-compose --version 
docker-compose version: 1.11.2

3.4 Compose其他安装方法

3.4.1 pip

# pip install docker-compose

3.4.2 Install as a container

# curl -L https://github.com/docker/compose/releases/download/1.11.2/run.sh > /usr/local/bin/docker-compose 
# chmod +x /usr/local/bin/docker-compose

4.	Compose 配置

Compose配置文件用的yml格式（docker-compose.yml），docker规定了一些指令，使用它们可以去设置对应的东西，它主要分为了3个区域：
services：是服务，在它下面可以定义应用需要的一些服务，每个服务都有自己的名字、使用的镜像、挂载的数据卷、所属的网络、依赖哪些其他服务等等；
networks：是应用的网络，在它下面可以定义应用的名字、使用的网络类型等等；
volumes：是数据卷，在它下面可以定义的数据卷（名字等等），然后挂载到不同的服务下去使用；

4.1 Service

先创建一个文件夹beck-docker，并在里面新建docker-compose.yml文件，然后使用version指定一下compose使用的版本号。

4.1.1 定义服务

在应用里需要定义的服务，可以放到services下面。比如，我们去定义一个dog的服务，使用nginx镜像，指定主机上的8080端口映射到容器中得80端口，也就是nginx的http的访问端口。

version: '2'
services: 
  dog: 
    image: nginx
    ports: 
      - "8080:80"
	  
以同样的方式，定义一个cat的服务，同样使用nginx镜像，然后指8081端口对应80端口。

version: '2'
services: 
  dog: 
    image: nginx
    ports: 
      - "8080:80"
  cat: 
    image: nginx
    ports:
      - "8081:80"
	  
需要注意的是，cat与dog要在同一个级别，还有ports是个数组，可以指定多个端口映射关系。

4.1.2 启动服务

定义好服务以后，在项目的目录启动这些服务，可以执行：
# docker-compose up
 
这样会启动，在compose文件下定义的所有服务。由于这是第一次启动这个服务，所以可以看到它是creating，也就是去创建相关的东西。首先会创建这个服务使用的网络，这里是叫做「beckdocker_default」的网络，然后是dog和cat的服务，这些网络和服务的名字，默认会加上一个前缀，由于在创建应用的时候没有指定名字，所以会默认使用项目目录的名字，后面还有一个数字的后缀。最后会有一个「Attaching to …」，将网络应用到服务上。
启动成功后，在浏览器访问一下。8080对应的是dog的服务，8081是cat的服务。
 
回到终端，可以看到服务的访问日志，日志的开头会有服务的名字，标志着日志是从哪个服务来的：
 
如果希望服务在后台运行，可以使用-d选项（也就是detach）：
# docker-compose up –d

4.1.3 服务的生命周期

查看正在运行的服务：
# docker-compose ps
停止一个服务：
# docker-compose stop [服务名]
如果后面不加服务名，会停止所有的服务。
启动某一个服务:
# docker-compose start [服务名]
如果后面不加服务名，会启动所有的服务。
查看服务运行的log:
# docker-compose logs -f
加上-f选项，可以持续跟中服务产生的log。
 
进入服务容器中:
# docker-compose exec dog bash
删除服务:
# docker-compose rm
 
注意这个docker-compose rm不会删除应用的网络和数据卷。查看一下网络，可以看到应用创建的网络「beckdocker_default」，如果要删除所有的这些，可以使用：
# docker-compose down
 
会提示我们删除网络 beckdocker_default。

4.2 Networks

网络决定了服务之间以及服务和外界之间如何去通信，在执行docker-compose up的时候，docker会默认创建一个默认的网络，创建的服务也会默认地属于这个默认网络。服务和服务之间，可以使用服务的名字进行通信。也可以自己创建网络，并将服务属于到这个网络之中，这样服务之间可以相互通信，而外界就不能够与这个网络中的服务通信，可以保持隔离性。
 
下面登录dog服务去连接cat服务，登录到cat服务连接dog服务：
 
可以通过服务的名称进行连接。
自定义网络
A. 在networks中先定义一个名为animal，类型为bridge的网络：
version: '2'
services:
  dog:
    image: nginx
    ports:
      - "8080:80"
  cat:
    image: nginx
    ports:
      - "8081:80"
networks:
  animal:
    driver: bridge
B. 让dog和cat服务使用这个网络：
version: '2'
services:
  dog:
    image: nginx
    ports:
      - "8080:80"
    networks:
      - "animal"
  cat:
    image: nginx
    ports:
      - "8081:80"
    networks:
      - "animal"
networks:
  animal:
    driver: bridge
C. 再增加一个叫pig的服务，使用默认网络，来体现于自定义网络的隔离性：
version: '2'
services:
  dog:
    image: nginx
    ports:
      - "8080:80"
    networks:
      - "animal"
  cat:
    image: nginx
    ports:
      - "8081:80"
    networks:
      - "animal"
  pig:
    image: nginx
    ports:
      - "8082:80"
    networks:
      - "default"
networks:
  animal:
    driver: bridge
	
D. 重新启动应用:
# docker-compose up –d
 
E. 登录cat服务，尝试去连接dog服务和pig服务：
 
因为cat与dog同在animal网络，所以可以通过名字连接，而pig在default网络中，所以不能。

4.3 Volumes

在compose文件中，还可以指定一些有名字的数据卷，让服务去使用。方法是：在与networks同级的地方，添加volumes，接着是数据卷的名字，下面使用driver去指定数据卷的类型。
 
定义好数据卷后，就将这个数据卷交给一个服务去使用。可以用volumes给服务指定需要使用的数据卷：
dog:
  ...
  volumes:
    - nest:/mnt
以dog服务为例，volumes下指定使用的数据卷，冒号左边是数据卷名称，冒号右边是挂载到的docker对应目录位置。接着给cat服务也添加同样的数据卷。
 
回到终端，执行docker-compose up -d，下面测试一下数据卷：
 
因为cat与dog服务都使用nest的数据卷，所以在dog中/mnt目录下创建的data1，在cat服务的/mnt目录下可以看到。
指定位置的数据卷
dog和cat都是一个web服务，现在我想将主机的某一个位置当做是服务的一个内容，那么我们可以去创一个指定位置的数据卷。首先可以在当前目录，创建./app/web文件夹，在里面创建个index.html。
编辑内容：
 
内容编辑好后，就给dog与cat服务指定数据卷，冒号左边是主机上的目录，冒号右边是服务内挂载的目录：
 
说明：/usr/share/nginx/html 目录是nginx默认主机的根目录，也就是nginx欢迎界面的目录。
重新启动一下docker-compose up -d，访问一下dog与cat服务：
 

<img src="/assets/image/test.png" alt="替代文本" title="标题文本" width="200" height = "100" />

