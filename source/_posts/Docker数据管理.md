---
title: Docker数据管理
date: 2017-03-07 13:25:24
categories: Docker
tags: [Docker]
comments: true
toc: true
---


## Docker数据管理

### 数据卷

数据卷是一个或多个容器专门指定绕过Union File System的特殊目录，为持续性或共享数据提供一些有用的功能。数据卷可以用来存储Docker应用的数据，也可以用来在Docker容器间进行数据共享。 使用Docker的数据卷，类似在系统中使用 mount 挂载一个文件系统

- 数据卷可以在容器间共享和重用
- 数据卷数据改变是直接修改的
- 对数据卷的更新，不会影响镜像
- 数据卷是持续性的，直到没有容器使用它们

#### 添加一个数据卷

使用 -v 选项添加一个数据卷，或者可以使用多次 -v 选项为一个 docker 容器运行挂载多个数据卷 

```shell
#加载数据卷到容器的/webapp目录
sudo docker run -d -P --name web -v /webapp training/webapp python app.py
```

#### 删除数据卷

数据卷是独立于容器，Docker不会在容器被删除后自动删除数据卷，也不会有垃圾回收机制来处理。如果要删除数据卷，可以在删除容器的时候，加个 -v 的选项。

#### 挂载一个主机目录作为数据卷

挂载是啥意思啊？答：挂载就是把设备放在一个目录下，用U盘例子理解一下哦~

可以直接挂载宿主机文件或目录(设备)到容器(目录)里，可以理解为目录映射，这样就可以让所有的容器共享宿主机数据，从而只需要改变宿主机的数据源就能够影响到所有的容器数据。 

挂载的数据默认为可读写权限。 :ro 为挂载的数据为只读

```shell
#-v后面的映射关系是"宿主机文件/目录:容器里对应的文件/目录"，其中，宿主机上的文件/目录是要提前存在的，容器里对应的文件/目录会自动创建。 
docker run -t -i --name test -v /src/webapp/1.txt:/opt/webapp/1.txt:ro docker.io/centos /bin/bash
docker run -t -i --name hqsb -v /src/webapp:/opt/webapp docker.io/centos /bin/bash
```

#### 查看数据卷的具体信息

```
docker inspect web
```

### 数据卷容器

数据卷容器，其实是一个正常的容器，专门用来提供数据卷供其它容器挂载的。

用户需要在多个容器之间共享一些持续更新的数据，最好用数据卷容器

```shell
#创建数据卷容器
docker run -d -v /dbdata --name dbdata training/postgres
#使用 --volumes-from 来挂载容器中的数据卷
docker run -d --volumes-from dbdata --name db1 training/postgres

docker run -d --name db3 --volumes-from db1 training/postgres
```

#### 利用数据卷容器来迁移数据

可以利用数据卷容器对其中的数据卷进行备份、恢复，以实现数据的迁移。 

1. ##### 备份

   首先利用ubuntu镜像创建了一个容器worker。使用--volumes-from dbdata参数来让worker容器挂载dbdata容器的数据卷(即dbdata数据卷),使用-v  $(pwd):/backup参数来挂载本地的当前目录到worker容器的/backup目录。worker容器启动后，使用了tar cvf  /backup/backup.tar /dbdata命令来将/dbdata下内容备份为容器内的/backup/backup.tar，即宿主主机当前目录下的backup.tar。 

   ```shell
   docker run --volumes-from dbdata -v $(pwd):/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata
   ```

2. ##### 恢复

   首先创建一个带有数据卷的容器dbdata2：

   ```shell
   docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
   ```

   然后创建另一个新的容器，挂载dbdata2的容器，并使用untar解压备份文件到所挂载的容器卷中：

   ```shell
   docker run --volumes-from dbdata2 -v $(pwd):/backup --name worker ubuntu tar xvf /backup/backup.tar
   ```

   