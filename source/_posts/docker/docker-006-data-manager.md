---
title: Docker Journal（六）Data Management and Volume
date: 2017-05-24 18:00:16
categories:
tags:
---

# 数据管理
## 一.数据卷(Data Volume)

数据卷是一个可供容器使用的特殊目录，有如下特性：

数据卷可以在容器之间共享和重用

数据卷修改会立即生效

数据卷的更新不会影响镜像

如果有容器使用数据卷，该卷会一直存在

在容器内创建数据卷

在使用docker run的命令时，使用 -v 标记可以在容器内创建一个数据卷，并且可以指定挂在一个本地已有的目录到容器中作为数据卷：

docker run -d --name app1 -it -v ${PWD}/webapp:/root/webapp ubuntu bash

注意：默认挂载的数据卷的权限是rw（可读写），如果要求ro（只读），则需要加上对应的ro参数，命令可改为：

docker run -d --name app1 -it -v ${PWD}/webapp:/root/webapp:ro ubuntu bash

下面我们一起来操作一下：

创建webapp目录，在目录下新建文件file，并在文件file中写入 “this is a file”。



echo ${PWD} 命令标识当前目录。



创建启动app1容器并挂载数据卷



进入容器找到root目录可查看到已挂载的数据卷。



数据卷目录与容器内目录有映射关系，所以不管是在容器内部修改数据卷还是在外部修改数据卷，相对应的数据卷都会发生改变。



## 二.数据卷容器

数据卷容器用于用户需要在容器间共享一些持续更新的数据，数据卷容器专门提供数据卷供其它容器挂载使用。

Example:

创建数据卷容器db1

docker run -d --name db1 -v /dbdata -ti ubuntu bash

创建容器db2与db1共享dbdata的数据

docker run -d --name db2 --volumes-from db1 -ti ubuntu bash



在容器db1和容器db2任意一个容器修改dbdata的内容，在两个容器内均生效



数据卷容器的删除：

如果删除了挂载的容器，数据卷并不会被自动删除，如果要删除一个数据卷，必须在删除最后一个还挂载它的容器时显示使用docker rm -v 命令指定同时删除关联的容器。在下图可看到即使删除atest，btest中仍然有fileA文件。



## 三.利用数据卷迁移容器

可以利用数据卷容器对其中的数据卷进行备份、恢复，以实现数据的迁移。

备份：

使用下面的命令来备份dbdata数据卷容器内的数据卷：

docker run --volumes-from dbdata -v ${PWD}:/backup --name worker ubuntu \tar cvf /backup/backup.tar /dbdata

说明：

利用ubuntu镜像创建一个容器worker。使用--volumes-from dbdata参数来让worker容器挂载dbdata的数据卷；使用${pwd}:/backup参数来挂载本地目录到worker容器的/backup目录。

worker启动后，使用tar命令将/dbdata下的内容备份为容器内的/backup/backup.tar。

创建dbdata数据卷容器并写入文件：fileA、fileB、fileC



执行备份命令创建备份tar包：



恢复：

如果恢复数据到一个容器，可以参照下面的操作。首先创建一个带有数据卷的容器dbdata2:

docker run -d -v /dbdata --name dbdata2 ubuntu /bin/bash

然后创建另一个新的容器，挂载dbdata2的容器，并使用tar命令解压备份文件到挂载的容器卷中即可：

docker run --volumes-from dbdata2 -v ${pwd}:/backup ubuntu tar xvf /backup/backup.tar

