---
title: Docker的镜像
date: 2017-03-07 13:25:24
categories: Docker
tags: [Docker]
comments: true
toc: true
---



|    子命令分类    |                            子命令                            |
| :--------------: | :----------------------------------------------------------: |
|  Docker环境信息  |                        info、version                         |
| 容器生命周期管理 | create、exec、kill、pause、restart、rm、run、start、stop、unpause |
|   镜像仓库命令   |              login、logout、pull、push、search               |
|     镜像管理     |     build、images、import、load、rmi、save、tag、commit      |
|   容器运维操作   | attach、export、inspect、port、ps、rename、status、top、wait、cp、diff、update |
|   容器资源管理   |                       volume、network                        |
|   系统日志信息   |                    events、history、logs                     |

## Docker的镜像

#### 拉取镜像

Docker运行容器前需要本地存在对应的镜像，如果镜像不存在本地，Docker会从镜像仓库下载（默认是Docker Hub 公共注册服务器中的仓库）

```
docker pull [选项] [docker Registry地址]<仓库名>:<标签>
```
Docker Registry地址：<域名/IP>[:端口号]，默认地址为Docker Hub
仓库名：<用户名>/<软件名>。对于Docker Hub，默认为library，也就是官方的镜像。

#### 列出镜像

```shell
//-a 显示包括中间层镜像在内的全部镜像
docker images [-a] [image_name]:[tag]
//filter过滤参数 --filter/-f
docker images -f since=[image_name]  #镜像版本之前的，用before

#格式化显示
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

#### 删除本地镜像

```shell
#镜像可以是 镜像短ID（去前3个字符以上）、镜像长ID、镜像名或者镜像摘要，(镜像名:Tag)
docker rmi [选项] <镜像1> [<镜像2> ...]
```

#### Commit定制镜像

```shell
//构建镜像，commit命令，不推荐
//将容器的存储层保存下来成为镜像
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```

#### Dockerfile定制镜像

##### 基础命令

1、FROM指定基础镜像

2、 RUN执行命令

- shell格式：RUN <命令>  ，直接在命令行输入命令

```shell
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

- exec格式：RUN ["可执行文件"，"参数1"，"参数2"]，类似函数调用



举个栗子：

```shell
FROM debian:jessie
RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

RUN命令中使用&&将各个所需命令串联起来，支持Shell类 的行尾添加 \ 的命令换行方式，以及行首#进行注释的格式

在Dockerfile 文件所在目录执行，构建镜像。RUN指令首先启动一个容器，执行所要求的命令，提交一个新的镜像中，随后删除所用到的容器

```shell
docker build -t xx:v3 . 
```

镜像构建上下文

	在上面的命令中，我们发现在docker build的最后面有一个 .   表示当前目录。但是这不是在指定Dockerfile所在的路径。==因为在默认情况下，如果不指定Dockerfile，会将上下文目录下名为Dockerfile的文件作为Dockerfile。==再举个栗子吧：我们不能COPY ../xx/yyy 或者 COPY /opt/xx/yyy。因为这些路径已经是超出上下文的范围了，我们是将当前目录指定为上下文目录的。（Dockerfile文件名不需要为Dockerfile，也不要求位于上下文目录中，可以用 -f 来指定文件）
	
	为什么我们需要上下文这个概念呢？因为docker客户端的命令通过Docker Remote API 和Docker引擎进行交互，一切都是使用远程调用形式在服务端（Docker引擎）完成。所以在我们构建的时候，用户会指定构建镜像上下文的路径。



3、COPY复制文件





## Docker的容器

#### 新建、启动容器

```shell
//新建并启动容器
//-i标志保证容器中的STDIN开启，-t分配一个伪tty终端,-d为后台运行，
//-p为指定端口映射，宿主机:容器，/bin/bash启动一个Bash shell
docker run --name yyy(容器名) [-d] -i -t xxxx(镜像名) /bin/bash

//进入容器
docker exec -it container_name /bin/bash
```

```
// -a 查看全部容器，包括停止和运行，-q 查看容器ID
docker ps -a 
```

```shell
//启动已经停止的容器
docker start container_name[或者container_id]
//重启容器
docker restart container_name[或者container_id]
//删除容器
docker rm [-f] 
docker rm $(docker ps -a -q)
```

#### 导入导出容器

```shell
#导出容器快照
docker export container_ID > xxx.tar
#导入容器快照
cat xxx.tar | docker import - xxx/yyy:v1.0
docker import url
```

docker load 和 docker import的区别：

docker load 是导入镜像存储文件到本地镜像库。

容器快照文件将丢弃所有的历史纪录和元数据信息（仅保存容器当时的快照状态），而镜像存储文件将保存完整纪录，体积也很大。

#### 容器互联

容器的连接系统是除了端口映射外，另一种跟容器中应用交互的方式。该系统会在源和接受容器之间创建一个隧道，接受肉国企可以看到源容器指定的信息。

使用--link参数使容器之间安全进行交互，--link name:alias，name为要链接的容器的名字，alias是这个链接的别名

```shell
docker run -d --name db training/postgres
docker run -d -P --name web --link db:db training/webapp
#用docker ps查看容器的连接
在自定义命名的容器，有db和web，在db容器的names列有db,web/db，表示web容器链接到db容器，web容器被允许访问db容器信息。
上面这个做法，可以避免db容器暴露数据库端口到外部网络上。
```

```shell
#2种方式为容器公开连接信息
第一种：环境变量，用env命令查看web容器的环境变量
docker run --rm --name web2 --link db:db training/webapp env
第二种：更新/etc/hosts文件
```



## Docker的仓库

#### Docker Hub

```shell
#登录，输入用户名、密码、邮箱来完成注册和登录。注册完成后，本地用户目录.dockercfg保存用户的认证信息
docker login 
#-s N：显示评价为N星以上的镜像
docker search [-s N] image_name
#下载镜像到本地
docker pull
docker push
```

#### 私有仓库

docker-registry是官方提供的工具，可以用于构建私有的镜像仓库。

**从容器运行Registry**

```shell
#默认情况下，仓库会被创建在容器的/temp/registry下，-v指定将镜像文件存放在本地的指定路径
docker run -d -p 5000:500 -v /opt/data/registry:/tmp/registry registry
```

