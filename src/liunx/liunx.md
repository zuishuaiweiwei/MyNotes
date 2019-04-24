# liumx

## 1、常用命令

 

| 产看内核版本          | uname -r           |
| --------------------- | ------------------ |
| **查看liunx系统版本** | **cat /etc/issue** |
|                       |                    |
|                       |                    |
|                       |                    |
|                       |                    |
|                       |                    |
|                       |                    |
|                       |                    |
|                       |                    |

------



## 2 常用软件安装

 ### 1、 mysql

### 2、redis

### 3、rabbitmq

### 4、tomcat

### 3 、docker

```
1、centos版本在6.5以上 内核版本在3.1以上

yum install https://get.docker.com/rpm/1.7.1/centos-6/RPMS/x86_64/docker-engine-1.7.1-1.el6.x86_64.rpm

```

### 4、elasticsearch

​	官网下载安装包，解压

```shell
1、创建普通用户组
groupadd elsearch                                      新建组
useradd -g elsearch elsearch                           新建用户，赋予组
passwd elsearch                                        设置用户新密码
chown -R elsearch:elsearch /opt/elasticsearch/          将目录权限赋给用户
```

```shell
2.需要将JVM虚拟机内存设置修改	
vim /elasticsearch-7.0.0/config/jvm.options
修改内容
-Xms256m
-Xmx256m
```

​	然后在bin目录下使用elsearch用户直接 ./elasticsearch是可以启动的，不过绑定的端口是127.0.0.1 只有本机访问，需要修改配置文件

```shell
2、修改elsearch配置文件
vim /elasticsearch/config/elasticsearch.yml 
修改内容
network.host: ip地址 
or
network.host: 0.0.0.0
```

​	然后启动报错

（1）、max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]

```shell
Root用户在文件/etc/security/limits.confg添加配置

* soft nofile 65536
* hard nofile 131072
```

（2）、 max number of threads [1024] for user [elsearch] is too low, increase to at least [4096]   

```
Root用户在文件/etc/security/limits.confg添加配置
* soft nproc 2048
* hard nproc 4096
或者是
使用Root用户修改文件/etc/security/limits.d/90-nproc.conf 配置
*  soft    nproc  1024 
改为  
*  soft    nproc  4096 
或者是两个都改
改完source和. 命令不起作用 需要重新启动
```

（3）、max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```
使用Root用户在文件 /etc/sysctl.conf 添加配置
vm.max_map_count=655360
执行命令，使配置生效
sysctl -p
```

（4）、the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured

```
修改elsearch配置文件
vim /elasticsearch/config/elasticsearch.yml 

将    #cluster.initial_master_nodes: ["node-1", "node-2"] 
修改为 cluster.initial_master_nodes: ["node-1"]
```

​	然后就可以启动了



------



## 3 其他

### 1、升级内核  

```shell
1、检查当前版本
uname -a      
2、导入public key
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org            
3、安装elrepo
rpm -Uvh http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm    
4、安装lt
yum --enablerepo=elrepo-kernel install kernel-lt -y  
5、编辑grub.conf文件，修改Grub引导顺序
vim /etc/grub.conf         
        default改为0

```

### 2、设置开机启动服务

```shell
chkconfig docker on 
```

### 3、关闭防火墙

```shell
打开防火墙：service iptables start
关闭防火墙：service iptables stop
重启防火墙：service iptables restart
查看防火墙状态：service iptables status
上面都是临时关闭防火墙 因为防火墙都是开机自启动的 
永久开启防火墙：chkconfig iptables on
永久关闭防火墙：chkconfig iptables off
查看防火墙配置文件vi /etc/sysconfig/iptables

```

### 4、设置环境变量

```shell
vi /etc/profile        设置环境变量
source /etc/profile    修改立即生效 or . /etc/profile
```



