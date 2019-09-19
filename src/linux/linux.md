# liunx

## 一、常用命令

 

| 产看内核版本              | uname -r                                    |
| :------------------------ | ------------------------------------------- |
| **查看liunx系统版本**     | **cat /etc/issue**                          |
| stty -a                   | 查看所有快捷键                              |
| df -h                     | 查看硬盘使用状态                            |
| free                      | 查看内存使用情况                            |
| ntpdate asia.pool.ntp.org | 同步时间                                    |
| echo $?                   | 查看上一条命令是否正确执行                  |
| **&>/dev/null**           | 不让打印命令结果重定向到垃圾箱              |
| netstat -tuln             | 查看正在监听的tcp和udp端口                  |
| nmap -sT ip               | 扫描ip地址的主机开启的Tcp端口，是否正常响应 |

------

### 1、cut

​	列提取命令，没有办法用空格用作分隔符

| 参数         | 作用                            |
| ------------ | ------------------------------- |
| -d           | 指定分隔符，默认为制表符        |
| -f           | 制定列，                        |
| --complement | 提取指定字段之外的列 和 -f 配合 |
| -c           | 指定字符数   基本上不用         |

​	

```
grep "root" /etc/passwd | cut -d ":" -f 1

eq

grep "/bin/bash" /etc/passwd | grep -v "root" | awk 'BEGIN{FS=":"} {print $1}'

```

### 2、文本三剑客

#### 1）grep

​	行提取命令，可以和正则配合使用

| 参数         | 作用                             |
| ------------ | -------------------------------- |
| -n           | 显示行号                         |
| -v           | 取反                             |
| -o           | 仅显示你需要的东西，默认打印正行 |
| -i           | 忽略大小写                       |
| -c           | 统计出现的次数                   |
| --color=auto | 过滤字段加颜色                   |

#### 2）awk

​	awk是一门编程语言，在liunx中 主要用来输出指定列，因为cut可能处理不了的一些空格问题，awk就可以解决

​	格式： awk '条件1{动作1}  条件2{动作2}'    条件可以没有 至少一个动作

| 内置参数 | 作用                                       |
| -------- | ------------------------------------------ |
| $0       | 表示整行数据，awk是一行一行读取数据的      |
| $n       | n为数字 表示第几列                         |
| NR       | 第一行                                     |
| NF       | 这行数据总共几列                           |
| FS       | 指定分隔符 ，一般和BEGIN连用 BEGIN{FS=":"} |



```shell
df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        19G   12G  5.6G  69% /
tmpfs           488M   72K  488M   1% /dev/shm
/dev/sda1       190M  101M   75M  58% /boot


df -h | awk '{print $n}'
n是第几列

df -h |grep "/dev/sda3" | awk ' {print $5} ' | cut -d "%" -f 1
查看根分区的使用大小

## 注意里面的制表符为双引号
df -h | awk '{print $1 "\t" $5}'  
Filesystem	Use%
/dev/sda3	69%
tmpfs	0%
/dev/sda1	58%
#### 前面加上BEGIN 在打印之前先处理BEGIN里面的命令，END则相反
df -h | awk 'BEGIN{print 1111111}{print $1 "\t" $5}' 

## 如果想让awk识别字符串 要加上// ~ 是否包含 ！~是否不包含
awk '$1 ~ /root/ {print}' /etc/passwd  

# {FS=":"} 指定分隔符  BEGIN 读取数据之前就告诉awk  否则第一行还是按照默认的分隔符
grep "/bin/bash" /etc/passwd | grep -v "root" | awk 'BEGIN{FS=":"} {print $1}'

#$3==500 指定打印第三列等于500才打印第一列的条件 
grep "/bin/bash" /etc/passwd | grep -v "root" | awk 'BEGIN{FS=":"} $3==500 {print $1}

# {}里面的内置参数不加引号 字符串和 \t 需要加引号 "\t 行号：" eqs "\t""行号："
grep "/bin/bash" /etc/passwd | grep -v "root" | awk 'BEGIN{FS=":"} {print $1 "\t 行号：" NR  "\t 列数 ：" NF}
wei	 行号：1	 列数 ：7
mysql	 行号：2	 列数 ：7
elsearch	 行号：3	 列数 ：7

```

#### 3）sed

​	sed用来将数据进行选取，替换，删除，新增

​	sed [选项] '[动作]' 文件名

| 选项 | 说明                                                     |
| ---- | -------------------------------------------------------- |
| -n   | sed一般会把所有数据输出出来，-n 只会把经过处理的数据输出 |
| -e   | 允许对输入数据应用多条sed命令编辑                        |
| -f   | 从sed脚本中读取sed操作，和awk -f非常类似                 |
| -r   | 支持扩展正则表达式                                       |
| -i   | 用sed的修改结果直接修改文件，不是输出到屏幕              |



| 动作 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| a    | 追加，在当前行后添行。添加多行时，需要加 \                   |
| c    | 行替换，用c后面的字符串替换原本数据行、替换多行时，除最后一行外，每行末尾都需要用\表示数据没有完结，不添加一行则不写 \ |
| i    | 插入，在当前行前插入一行或多行、插入多行时，除最后一行外，每行末尾都需要用\表示数据没有完结,不添加一行则不写 \ |
| d    | 删除，删除指定行                                             |
| s    | 字符串替换 格式为：s\旧字符串\新字符串\g （和vim的格式一样） |
| p    | 打印指定行                                                   |



```shell
# 打印第二行数据
sed -n "2p"  /etc/passwd

#删除2，3，4行 ，但是没有修改文件本身 ，只是屏幕输出时删除想·，想修改文件加上 -i 
sed "2,4d" filePath 

#在第二行后面添加一行xxx，没有-i，还是没有修改文件本身
sed "2a xxx" filePath 

#在第二行前边添加一行xxx，没有-i，还是没有修改文件本身
sed "2i xxx" filePath 

#在第二行前边添加两行，自己手动换行，防止自动换行出错，没有-i，还是没有修改文件本身
sed "2i xxx \ 
ccc" filePath 

# 第二行替换为xxx
sed "2c xxx" filePath 

# -e 执行多条命令 用 ; 分割。第二行替换为xxx并且删除第三行
sed -e "2c xxx; 3d" filePath 

#全文本匹配gg替换为 he
sed "s/gg/he/g" filePath


```



### 3、sort

​	sort是字符串处理命令

| 选项 | 说明                             |
| ---- | -------------------------------- |
| -f   | 忽略大小写                       |
| -b   | 忽略每行前边的空白               |
| -n   | 以数值型进行排序，默认使用字符串 |
| -r   | 反向排序                         |
| -u   | 删除重复行 就是uniq命令          |
| -t   | 指定分隔符                       |



```shell
#默认按照每行开头的第一个字符来排序
sort filePath

#排序顺序取反
sort -r filePath

#指定字段来排序 -t指定分隔符 -k指定第几个字段 （但是如果是数字，还是按照字符形式来排序，按照数字排序加上-n）
sort -t ":" -k 3,3 filePath
sort -n -t ":" -k 3,3 filePath

```



### 4、uniq

​	取消重复行 和 sort -u选项一样



```
uniq [选项] filePath
-i 忽略大小写
```

### 5、wc

​	统计命令

```
wc [选项] filePath

-l 只统计行数
-w 只统计单词数
-m 只统计字符数
```



### 6、ln

​	硬链接，在指定目录新建文件 指向同一个节点，不占用重复的磁盘空间

​	-s 软链接 其大小为路径所包含的字符个数，引用了源文件的路径，删除源文件失效，硬链接则不会

### 7、rpm

```shell
# 直接安装
rpm -ivh your-package         
# 忽略报错，强制安装
rpmrpm --force -ivh your-package.rpm 
# 查询
rpm -ql name        
# 卸载
rpm -e name        
# 列出所有安装过的包
rpm -qa 
# 输出软件包的全名称
rpm -q name
# 列出软件所有文件安装的位置
rpm -ql tree 
```

### 8、tail

​	就是把某个档案文件的最后几行显示到终端上，假设该档案有更新，tail会自己主动刷新，确保你看到最新的档案内容。

```shell
# 监视filename文件的尾部内容（默认10行，相当于增加参数 -n 10），刷新显示在屏幕上。退出，按下CTRL+C
# -f 是继续监视文件 实时刷新
# -n 指定行数
tail -f filename
# 逆序显示filename最后10行。
tail -r -n 10 filename
# 显示filename的后6行
tail -6 filename
```

### 9、head

​	和tail相反，显示文件的开头部分

```shell
# 显示filename的前6行
head -6 filename

```



## 二、 常用软件安装

 ### 1、 mysql

### 2、redis

### 3、rabbitmq

### 4、tomcat

### 3 、docker

```shell
#centos版本在6.5以上 内核版本在3.1以上
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

```shell
#使用Root用户在文件 /etc/sysctl.conf 添加配置
vm.max_map_count=655360
#执行命令，使配置生效
sysctl -p
```

（4）、the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured

```shell
#修改elsearch配置文件
vim /elasticsearch/config/elasticsearch.yml 

将    #cluster.initial_master_nodes: ["node-1", "node-2"] 
修改为 cluster.initial_master_nodes: ["node-1"]
```

​	然后就可以启动了

### 5、python

​	

```shell
#安装依赖
yum install gcc -y
yum install zlib -y
yum install zlib-devel -y
yum install openssl-devel -y
#下载压缩包、解压
cd /usr/local/
wget https://www.python.org/ftp/python/3.4.4/Python-3.4.4.tar.xz
tar -xvJf Python-3.4.4.tar.xz
#编译安装
cd /usr/local/Python-3.4.4
./configure prefix=/usr/local/python3
make && make install
#设置环境变量
```

### 6、nginx

```shell
#
vi  /etc/yum.repos.d/nginx.repo 
#
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/6/$basearch/   
gpgcheck=0
enabled=1
#
yum install nginx
```



------



## 三、 其他

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



### 5、设置命令别名

```shell
vim /root/.bashrc

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias vi='vim'

```

### 6、bash的快捷键

| 命令   | 作用                   |
| ------ | ---------------------- |
| ctrl+A | 光标移动到命令行开头   |
| ctrl+E | 光标移动到命令行结尾   |
| ctrl+C | 强行终止当前命令       |
| ctrl+Z | 暂停当前命令，放入后台 |
| ctrl+L | 相当于clear            |
| ctrl+U | 剪切光标位置之前的命令 |
| ctrl+K | 剪切光标位置之后的命令 |
| ctrl+Y | 复制剪切的命令         |

### 7、输出重定向

​	键盘是标准输入，显示器是标准输出

​	把信息保存到文件里叫输出重定向

```tex
> 覆盖文件 
>> 追加文件
命令 >> abc 2>&1 
命令 &> abc &>> abc
不管是正确信息还是错误信息都会保存在一个文件里
命令 >> abc 2>>error
正确输出到abc中，错误输出到error中
```

### 8、bash中的特殊符号

| 符号 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| ‘ ’  | 单引号，在里面的内容是普通字符串，特殊含义的符号失去意义     |
| “ ”  | 双引号，里面除了 "$"(调用系统变量) “`”(引用命令)  “\”(转义符) 有特殊含义 |
| ``   | 反引号，告诉系统里面的内容是系统命令。和$()的作用一样，推荐使用$() |
| $()  | 和反引号的作用一样                                           |
|      | 小括号，里面的命令是在子shell中执行 ，相当于一次分支，不影响父shell的内容 |
| {}   | 大括号，在当前shell执行命令，用的不多 ，直接写命令效果一样，第一个命令和{要有空格，最后的命令要有； |
| []   | 用于变量测试                                                 |
| #    | 行注释                                                       |
| $    | 取系统变量                                                   |
| \    | 转义符                                                       |

### 9、修改启动级别

​	

```shell
vi /etc/inittab 

id:3:initdefault:

0：关机
1：单用户模式
2：多用户无网络模式
3：命令行模式
5：GUI图形化界面模式
6：重启

不能设置为0 和 6
```

### 10、bash字体

```shell
export PS1="\[\033[1;36m\]\u\[\033[1;32m\][\w]\[\033[1;33m\]->\[\033[0m\]"
```



## 四、成功登陆时加载的系统文件

### 1、/etc/profile 

​	这个文件是用户登陆时成功后，加载的文件，里面是一些系统变量

​	里面会调用其它系统需要加载的配置文件，下面的代码是调用过程，依次调用/etc/profile.d/下以.sh结尾的文件

```shell
for i in /etc/profile.d/*.sh ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . "$i"
        else
            . "$i" >/dev/null 2>&1
        fi
   fi
done

```

### 2、/etc/profile.d/lang.sh

​	这个文件在开机会加载，主要配置环境信息,会加载默认的语言配置文件/etc/sysconfig/i18n

```shell
for i in /etc/profile.d/*.sh ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . "$i"
        else
            . "$i" >/dev/null 2>&1
        fi
   fi
done

```

### 3、/etc/sysconfig/i18n

​	里面就是系统默认的语言环境

### 4、~/.bash_profile

​	主要是加载了~/.bashrc文件

```shell

# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH
            
```

### 5、~/.bashrc

​	主要设置了一些别名，然后加载/etc/bashrc

```shell

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias vi='vim'
alias grep='grep --color=auto'
# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

```

### 6、/etc/bashrc

​	加载流程的最后一步，设置了PS1的值，设定登陆提示符的样子

​	如果在不用登陆的情况下切换到其他用户，会在这个文件加载没有登陆情况下的

```shell
if [ "$PS1" ]; then
  if [ -z "$PROMPT_COMMAND" ]; then
    case $TERM in
    xterm*)
        if [ -e /etc/sysconfig/bash-prompt-xterm ]; then
            PROMPT_COMMAND=/etc/sysconfig/bash-prompt-xterm
        else
            PROMPT_COMMAND='printf "\033]0;%s@%s:%s\007" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}"'
        fi
        ;;
    screen)
        if [ -e /etc/sysconfig/bash-prompt-screen ]; then
            PROMPT_COMMAND=/etc/sysconfig/bash-prompt-screen
        else
            PROMPT_COMMAND='printf "\033]0;%s@%s:%s\033\\" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}"'
        fi
        ;;
    *)
        [ -e /etc/sysconfig/bash-prompt-default ] && PROMPT_COMMAND=/etc/sysconfig/bash-prompt-default
        ;;
      esac
  fi
  # Turn on checkwinsize
  shopt -s checkwinsize
  [ "$PS1" = "\\s-\\v\\\$ " ] && PS1="[\u@\h \W]\\$ "

```

## 五、其他文件

### 1、~/bash_logout

​	注销时会执行的文件，里面可以放置一些备份资料的命令

### 2、~/bash_history

​	放置历史命令的文件

## 六、shell有关文件

### 1、/etc/issue

​	里面是在本地终端的提示信息，只对本地终端有效，远程登陆无用

```shell
CentOS release 6.8 (Final)
Kernel \r on an \m \l

```

### 2、/etc/issue.net

​	这个文件是远程登陆时的提示信息，内容和issue一样

​	如果需要这个文件生效，需要修改 /etc/ssh/sshd_config 文件,不过不识别转义的信息

```shell
打开注释，添加文件位置
Banner /etc/issue.net
```

### 3、/etc/motd

​	这个是登陆之后显示的欢迎信息，里面可以写提示的信息

## 七、shell编程

### 条件判断

#### 1）文件条件判断

| 测试选项 | 说明(都是先判断文件是否存在) |
| -------- | ---------------------------- |
| -b       | 是否为块设备文件             |
| -c       | 是否为字符设备文件           |
| **-d**   | 是否为目录文件               |
| **-e**   | 单纯的是否存在               |
| **-f**   | 是否为普通文件               |
| **-L**   | 是否为符号链接文件           |
| -p       | 是否为管道文件               |
| -s       | 是否不是空文件               |
| -S       | 是否为套接字文件             |



```shell
# 判断文件存不存在
# 推荐使用
#[] 两边都要有空格
[ -e filePath ] 
or
test -e filePath
# 查看上一条命令是否正确执行，正确返回0 错误非0
echo $? 
# 用另外一条命令查看结果不方便
#============
[ -e filePath ] && echo yes || echo no 
```



#### 2） 文件权限条件判断

| 测试选项 | 说明 (都是先判断文件是否存在) |
| -------- | ----------------------------- |
| -r       | 是否有读权限                  |
| -w       | 是否有写权限                  |
| -x       | 是否有执行权限                |
| -u       | 是否有SUID权限                |
| -g       | 是否有SGID权限                |
| -k       | 是否有SBIT权限                |



#### 3） 两个文件比较

| 测试选项            | 说明 (都是先判断文件是否存在)                                |
| ------------------- | ------------------------------------------------------------ |
| 文件1 **-nt** 文件2 | 文件1的修改时间是否比文件2 新                                |
| 文件1 **-ot** 文件2 | 文件1的修改时间是否比文件2 旧                                |
| 文件1 **-ef** 文件2 | 文件1和文件2的Inode号是否一致，是否为同一个文件，用于判断硬链接 |



#### 4）整数之间比较

| 测试选项          | 说明         |
| ----------------- | ------------ |
| 整数1  -eq  整数2 | 是否相等     |
| 整数1  -ne  整数2 | 是否不相等   |
| 整数1  -gt  整数2 | 是否大于     |
| 整数1  -lt  整数2 | 是否小于     |
| 整数1  -ge  整数2 | 是否大于等于 |
| 整数1  -le  整数2 | 是否小于等于 |



#### 5）字符串判断

| 测试选项           | 说明       |
| ------------------ | ---------- |
| -z 字符串          | 是否为空   |
| -n 字符串          | 是否非空   |
| 字符串1 == 字符串2 | 是否相等   |
| 字符串1 != 字符串2 | 是否不相等 |

#### 6）多重判断

| 测试选项       | 说明 |
| -------------- | ---- |
| 判断1 -a 判断2 | &&   |
| 判断1 -o 判断2 | \|\| |
| !判断          | 取反 |

### 流程控制

#### 1）if

​	

```shell
#单分支if
if [ 判断条件 ]；then
		做的事情		
fi
## then写到判断条件后面需要加 ； 
#判断条件和[] 两边有空格
或者写成这样
if [ 判断条件 ]
	then
		做的事情		
fi
#例子 如果根分区到百分之60就报警
#！/bin/bash
aa=$(df -h | grep "/dev/sda3" | awk '{print $5}'| cut -d "%" -f1)

if [ "$aa" -ge 60 ]
        then
                echo "error"
fi
```

```shell
#双分支if
if [ 条件 ] 
	then
		todo
	else
		todo
fi
```

```shell
#多分支if
if [ 条件 ]
	then
		todo
	elif[ 条件 ]
		then
			todo
	else[ 条件 ]
		then
			todo
fi

```



```shell
###四则运算计算机 shell版本
# if格式没有记清楚
# if里面的 == 两边需要有空格
# echo 输出运算的内容 $(($num1 + $num2)) 
# shell 中的数值运算  $(()) 

#!/bin/bash
read -t 30 -p "请输入第一个数字: " num1
read -t 30 -p "请输入运算符[+|-|*|/]: " opt
read -t 30 -p "请输入第二个数字: " num2
#echo $opt
#echo $num2
if [ -n "$num1" -a -n "$num2" -a -n "$opt" ]
        then
                test1=$(echo $num1 | sed 's/[0-9]//g')
                test2=$(echo $num2 | sed 's/[0-9]//g')
                if [ -z "$test1" -a -z "$test2" ]
                        then
                                if [ "$opt" == '+' ]
                                        then
                                                echo "this is +"
                                                echo $(($num1 + $num2))
                                                #exit 0
                                        elif [ "$opt" == '-' ]
                                                then
                                                        echo "this is -"
                                                        echo $(($num1 - $num2))
                                        elif [ "$opt" == '*' ]
                                                then
                                                        echo "this is *"
                                                        echo $(($num1 * $num2))
                                        elif [ "$opt" == '/' ]
                                                then
                                                        echo "this is /"
                                                        echo $(($num1 / $num2))
"count.sh" 43L, 928C                                                                                                      28,6-41       Top

```

#### 2）case

​	

```shell
case 变量 in
	"值1")
		todo
		;;
	"值2")
		todo
		;;
	*)
		todo
	;;
esac
```



#### 3）for

```shell
for 变量 in 值1 值2 值3
	do
		todo
	done
```



```shell
for ((i=1;i<=100;i=i+1))
	do
		todo
	done
```

#### 4）while

```shell
while [ 条件 ]
	do
		todo
	done
```

#### 5）until

​	和while是反着的 ，只要条件不成立就执行

```shell
until [ 条件 ]
	do
		todo
	done
```

#### 6）exit

​	特殊流程控制语句

​	shell脚本读到exit就退出脚本，不再继续执行

​	exit可以指定返回值 echo $?得到就是自己指定的返回值

#### 7）break

​	跳出整个当前循环

#### 8）continue

​	跳出这次循环，开始下一次循环



#### 6）简单判断合法IP

```shell
#!/bin/bash

cd /usr/local/mysh
## 粗略过滤的有效IP写入一个文件
$(cat ip.text | grep "[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}" > test1.txt)

aa=$(cat test1.txt| wc -l)
echo "第一次过滤有效行数 = $aa"
for i in $(cat test1.txt)
        do
         
                echo "遍历的ip： $i"
                echo "$i" > test2.txt
                num1=$(cut -d "." -f1 test2.txt)
                num2=$(cut -d "." -f2 test2.txt)
                num3=$(cut -d "." -f3 test2.txt)
                num4=$(cut -d "." -f4 test2.txt)


                #echo "$num1 : $num2 : $num3 : $num4"
                if [ "$num1" -lt 0 -o "$num1" -gt 256 ]
                        then
                        continue
                fi

                 if [ "$num2" -lt 0 -o "$num2" -gt 256 ]
                        then
                        continue
                fi

                 if [ "$num3" -lt 0 -o "$num3" -gt 256 ]
                        then
                        continue
                fi

                 if [ "$num4" -lt 0 -o "$num4" -gt 256 ]
                        then
                        continue
                fi
                echo "有效的ip $i"

        done

```

