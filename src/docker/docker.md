# docker

## 1、简介

**Docker**是一个开源的应用容器引擎；是一个轻量级容器技术；

Docker是一个虚拟软件容器，以往开发人员开发结束以后，将代码交给运维工程师，运维人员的机器和开发时的机器不是一台，可能会出现运行失败，Docker将开发时的整个环境，配置，代码一起交给运维，部署时就不会出现问题

## 2、核心概念

docker主机(Host)：安装了Docker程序的机器（Docker直接安装在操作系统之上）；

docker客户端(Client)：连接docker主机进行操作；

docker仓库(Registry)：用来保存各种打包好的软件镜像；

docker镜像(Images)：软件打包好的镜像；放在docker仓库中；

docker容器(Container)：镜像启动后的实例称为一个容器；容器是独立运行的一个或一组应用，容器是镜像的运行实例



仓库放镜像，下载镜像成容器





使用Docker的步骤：

1）、安装Docker

2）、去Docker仓库找到这个软件对应的镜像；

3）、使用Docker运行这个镜像，这个镜像就会生成一个Docker容器；

4）、对容器的启动停止就是对软件的启动停止；

## 3、安装、卸载Docker

```shell
yum install epel-release
yum install docker-io
如果报No package docker available执行
cd /etc/yum.repos.d
wget http://www.hop5.in/yum/el6/hop5.repo
再执行yum install docker-io
```

但是我自己安装完成之后查看版本和启动服务却显示错误

```
查看版本错误
Go version (client): go1.1.1
Git commit (client): -d
2019/07/02 15:40:37 dial unix /var/run/docker.sock: no such file or directory

启动服务错误
docker: unrecognized service
```

最后是这样安装成功的

```shell
yum install https://get.docker.com/rpm/1.7.1/centos-6/RPMS/x86_64/docker-engine-1.7.1-1.el6.x86_64.rpm
```

查看版本和启动服务正确

```shell
Client version: 1.7.1
Client API version: 1.19
Go version (client): go1.4.2
Git commit (client): 786b29d
OS/Arch (client): linux/amd64
Get http:///var/run/docker.sock/v1.19/version: dial unix /var/run/docker.sock: no such file or directory. Are you trying to connect to a TLS-enabled daemon without TLS?
```



```sehll
yum list installed | grep docker
yum -y remove docker-
```



## 4、Docker常用命令&操作

### 1）镜像操作

| 操作 | 命令                                              | 说明                                                     |
| ---- | ------------------------------------------------- | -------------------------------------------------------- |
| 检索 | docker  search 关键字  eg：docker  search redis   | 我们经常去docker  hub上检索镜像的详细信息，如镜像的TAG。 |
| 拉取 | docker pull 镜像名:tag                            | :tag是可选的，tag表示标签，多为软件的版本，默认是latest  |
| 列表 | docker images                                     | 查看所有本地镜像                                         |
| 删除 | docker rmi image-id                               | 删除镜像                                                 |
|      | docker rm container-id                            | 删除容器                                                 |
| 查看 | docker ps                                         | 查看正在运行的容器                                       |
| 停止 | docker kill container-id                          | 强制停止启动的容器                                       |
|      | docker stop container-id                          | 停止启动的容器                                           |
| 日志 | docker logs container-id                          | 查看容器日志                                             |
| 文件 | docker cp container-id：容器内路径 宿主机路径     | 容器里面的文件复制到宿主机上                             |
|      | docker run -it -v 宿主机文件路径：容器路径 镜像名 | 宿主机上的文件添加到容器里面                             |
| 详情 | docker inspect 容器                               | 查看容器详情                                             |

https://hub.docker.com/

​	**重要**

​	docker ps **-n 2**	查看最后启动的2个容器

​	docker ps **-a**	查看全部的容器（启动和停止）

​	docker ps **-l** 	查看最后启动的一个容器

​	docker logs  **-t** container-id  显示时间

​	docker logs  **-f** container-id  动态显示，不停追加

​	docker logs  **--tail 3** container-id  显示结尾行

​	启动交互式容器

```shell
docker run -it --name mycentos images-id
-it 指启动之后直接登陆到shell界面
退出有两种：输入exit 退出 容器也停止
		  crtl+q+p 退出 不停止
从新进入有两种： docker attach container-id		从新进入终端
			  docker exec container-id pwd 	 不进入终端 直接执行命令 返回结果
```

​	

### 2）容器操作

软件镜像（QQ安装程序）----运行镜像----产生一个容器（正在运行的软件，运行的QQ）；

步骤：

```shell
1、搜索镜像
[root@localhost ~]# docker search tomcat
2、拉取镜像
[root@localhost ~]# docker pull tomcat
3、根据镜像启动容器
docker run --name mytomcat -d tomcat:latest
4、docker ps  
查看运行中的容器
5、 停止运行中的容器
docker stop  容器的id
6、查看所有的容器
docker ps -a
7、启动容器
docker start 容器id
8、删除一个容器
 docker rm 容器id
9、启动一个做了端口映射的tomcat
[root@localhost ~]# docker run -d -p 8888:8080 tomcat
-d：后台运行
-p: 将主机的端口映射到容器的一个端口    主机端口:容器内部的端口

10、为了演示简单关闭了linux的防火墙
service firewalld status ；查看防火墙状态
service firewalld stop：关闭防火墙
11、查看容器的日志
docker logs container-name/container-id

更多命令参看
https://docs.docker.com/engine/reference/commandline/docker/
可以参考每一个镜像的文档
```

### 3）安装MySQL示例

```shell
docker pull mysql
```

错误的启动

```shell
[root@localhost ~]# docker run --name mysql01 -d mysql
42f09819908bb72dd99ae19e792e0a5d03c48638421fa64cce5f8ba0f40f5846

mysql退出了
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                           PORTS               NAMES
42f09819908b        mysql               "docker-entrypoint.sh"   34 seconds ago      Exited (1) 33 seconds ago                            mysql01
538bde63e500        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       compassionate_
goldstine
c4f1ac60b3fc        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       lonely_fermi
81ec743a5271        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       sick_ramanujan

//错误日志
[root@localhost ~]# docker logs 42f09819908b
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD；这个三个参数必须指定一个
```

正确的启动

```shell
[root@localhost ~]# docker run --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
b874c56bec49fb43024b3805ab51e9097da779f2f572c22c695305dedd684c5f
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b874c56bec49        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 3 seconds        3306/tcp            mysql01
```

做了端口映射

```shell
[root@localhost ~]# docker run -p 3306:3306 --name mysql02 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
ad10e4bc5c6a0f61cbad43898de71d366117d120e39db651844c0e73863b9434
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ad10e4bc5c6a        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 2 seconds        0.0.0.0:3306->3306/tcp   mysql02
```

几个其他的高级操作

```shell
docker run --name mysql03 -v /conf/mysql:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
把主机的/conf/mysql文件夹挂载到 mysqldocker容器的/etc/mysql/conf.d文件夹里面
改mysql的配置文件就只需要把mysql配置文件放在自定义的文件夹下（/conf/mysql）

docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
指定mysql的一些配置参数
```

### 4）镜像加速

```shell
docker pull registry.docker-cn.com/library/
```

## 5、常用软件run命令

### 1）tomcat

```shell
启动
docker run -d -p 8080:8080 --name mytomcat tomcat
进入控制台
docker exec -it  mytomcat /bin/bash
```

### 2）mysql

```
docker run -d -p 3306:3306 --name mymysql mysql
```

### 3）redis

```
docker run -d -p 6379:6379 --name myredis redis
```

### 4)elasticsearch

```shell
docker run -e ES_JAVA_OPTS="-Xms256m -Xm×256m" -d -p 9200:9200 -p 9300:9300 --name ES01 4ffc114372c8
```

### 5）rabbitmq

```shell
docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq rabbit
```

### 6）centos

```shell
docker run -it centos /bin/bash
```

## 6、镜像加速

```shell
vi /etc/sysconfig/docker
other_args="--registry-mirror=https://cw2dq19l.mirror.aliyuncs.com"
```

## 7、容器数据卷

​	数据卷是做数据持久化的，添加的文件是用挂载的方式添加的，主机和容器之间数据共享，容器退出，从新启动也同步

​	在镜像启动的时候添加文件

```shell
docker run -it -v 主机文件路径:容器文件路径 镜像名
```

![1562073698022](C:\Users\Administrator\Desktop\MyNotes\src\docker\images\1562073698022.png)

​	带权限的挂载

```shell
docker run -it -v 主机文件路径:容器文件路径:ro 镜像名
ro: read only
```

![1562074446216](C:\Users\Administrator\Desktop\MyNotes\src\docker\images\1562074446216.png)

### 1） dockerFile

​	docker里面的image类似于Java里的class文件，而dockerFile就是Java文件

​	dockerFile是image文件的描述文件，通过编写dockerFile文件，使用 docker build 命令，将dockerFile变成image，run的时候就可以挂载数据卷

​	demo

```shell
编写dockerFile文件
FROM centos
VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
CMD echo "finished.-------success1"
CMD /bin/bash
构建image
docker build -f /dockerFile/dockerfile -t ww/centos .   -f是路径 -t是命名空间 最后有个.不能忘

docker run -it ww/centos
```

![1562077501359](C:\Users\Administrator\Desktop\MyNotes\src\docker\images\1562077501359.png)

​	这个demo没有指定主机的映射路径，可以用inspect查看

​	容器间传递共享（--volumes-from）

```shell
docker run -it --name dc03 --volumes-from dc01 ww/centos
```

​	**容器之间的配置信息传递，数据卷的生命周期一直持续到没有容器使用它为止**