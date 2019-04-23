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

## 2 常用软件安装

 ### 1、 mysql

### 2、redis

### 3、rabbitmq

### 4、tomcat

### 3 、docker

```
1、centos版本在6.5以上 内核版本在3.1以上
sss
yum install https://get.docker.com/rpm/1.7.1/centos-6/RPMS/x86_64/docker-engine-1.7.1-1.el6.x86_64.rpm

```



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
查看防火墙状态：service iptables status
上面都是临时关闭防火墙 因为防火墙都是开机自启动的 
永久开启防火墙：chkconfig iptables on
永久关闭防火墙：chkconfig iptables off
查看防火墙配置文件vi /etc/sysconfig/iptables


```



