# Dubbo

## 一、dubbo概念

​	dubbo是一个分布式治理框架，providr和consumer都会在dubbo注册服务，consumer可以调用provider暴露出的接口。

## 二、dubbo和eureka的区别

​	 c 强一致性 a 可用性 p 分区容错性

​	**Zookeeper保证CP**

当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接down掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。但是zk会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30 ~ 120s, 且选举期间整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。

​	**Eureka保证AP**

​	Eureka看明白了这一点，因此在设计时就优先保证可用性。Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册或如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：

1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务

2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用)

3. 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

   因此， Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。

## 三、demo

​	创建一个空文件夹，用idea打开那个文件夹，在里面创建module，需要三个模块，一个provider、consumer、api。

​	api里面放置实体类和服务的接口。

### 1、创建服务提供者 user-service-provider

#### 1）导入pom

```xml
 <dependency>
            <groupId>com.wei.dubbo</groupId>
            <artifactId>dubbo-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>


        <!-- https://mvnrepository.com/artifact/com.alibaba/dubbo -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.2</version>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.9</version>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.10</version>
            <exclusions>
                <exclusion>
                    <artifactId>zookeeper</artifactId>
                    <groupId>org.apache.zookeeper</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.8.0</version>
        </dependency>
```

#### 2）创建配置文件provider.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">


    <context:component-scan base-package="com.wei.dubbo.*"></context:component-scan>
    
    <dubbo:application name="user-service-provider" ></dubbo:application>

    <dubbo:registry address="zookeeper://192.168.214.30:2181"></dubbo:registry>

    <dubbo:protocol port="20880" name="dubbo"></dubbo:protocol>

    <!--对外暴露的接口 -->
    <dubbo:service interface="com.wei.dubbo.service.UserService" ref="userService"></dubbo:service>
    <!--处理的实现类-->
    <bean name="userService" class="com.wei.dubbo.service.impl.UserServiceImpl"></bean>
</beans>
```

#### 3）创建UserServiceImpl 实现 api 模块下的 UserService

```java
package com.wei.dubbo.service.impl;

import com.wei.dubbo.entity.User;
import com.wei.dubbo.service.UserService;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;

/**
 * @author 为为
 * @create 7/26
 */
@Service
public class UserServiceImpl implements UserService{
        
    @Override
    public List<User> getUserList(){
        List<User> list = new ArrayList<>();
        User user1 = new User("weiwei", "wwww");
        User user2 = new User("jaiqi", "xxxx");
        list.add(user1);
        list.add(user2);
        return list;
    }
}
```

#### 4）创建启动类

```java
package com.wei.dubbo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.io.IOException;

/**
 * @author 为为
 * @create 7/26
 */
public class MainApplication {

    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("provider.xml");
        context.start();

        System.in.read();
    }
}
```

### 2、创建服务消费者 order-service-consumer

#### 1）导入pom

```xml
<dependency>
            <groupId>com.wei.dubbo</groupId>
            <artifactId>dubbo-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.alibaba/dubbo -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.2</version>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.9</version>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.10</version>
            <exclusions>
                <exclusion>
                    <artifactId>zookeeper</artifactId>
                    <groupId>org.apache.zookeeper</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.8.0</version>
        </dependency>
```

#### 2）创建consumer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.wei.dubbo.*"></context:component-scan>

    <dubbo:application name="order-service-consumer"></dubbo:application>

    <dubbo:registry address="zookeeper://192.168.214.30:2181"></dubbo:registry>

    <dubbo:reference id="userService" interface="com.wei.dubbo.service.UserService" ></dubbo:reference>
</beans>
```

#### 3）创建 OrderServiceImpl 实现 api模块下的 OrderServic

```java
package com.wei.dubbo.service.impl;

import com.wei.dubbo.entity.User;
import com.wei.dubbo.service.OrderService;
import com.wei.dubbo.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @author 为为
 * @create 7/26
 */
@Service
public class OrderServiceImpl implements OrderService{


    @Autowired
    private UserService userService;

    @Override
    public void getUserList(){
        List<User> users = userService.getUserList();
        for(User user:users){
            System.out.println(user);
        }
    }
}
```

#### 4）创建启动类

```java
package com.wei.dubbo;

import com.wei.dubbo.service.OrderService;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.io.IOException;

/**
 * @author 为为
 * @create 7/26
 */
public class MainApplication {

    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("consumer.xml");
        context.start();
        OrderService orderService = context.getBean(OrderService.class);
        System.out.println(orderService);
        orderService.getUserList();
        System.in.read();
    }
}
```

### 3、创建 api 模块

![1564191889843](C:\Users\Administrator\Desktop\MyNotes\src\dubbo\images\1564191889843.png)

## 四、安装、使用控制台、监控中心

### 1、安装监控中心

#### 1）、下载 dubbo-monitor-simple-xxx

#### 2）、修改conf/dubbo.properties

​	dubbo.jetty.port=8090 这个端口号不能和dubbo-admin的端口号一样

#### 3）、修改assembly.bin/start.sh

​	![1564196059687](C:\Users\Administrator\Desktop\MyNotes\src\dubbo\images\jvm参数.png)

​	默认是2g 修改为521m

#### 4）、访问监控中心主页

​	http://192.168.214.30:8090

#### 5）、修改项目配置文件

```xml
<dubbo:monitor protocol="registry"></dubbo:monitor>

使用这种报错 monitor在注册中心注册的地址是错的 所以获取的就是错的地址

<dubbo:monitor address="ip:xxxx"></dubbo:monitor>
建议手动指定
```

### 2、使用控制台

#### 1）下载控制台jar包

​	dubbo-admin-0.0.1-SNAPSHOT.jar

#### 2）运行，访问ip：7001

## 五、与Springboot整合

### 1、使用application.xml和注解结合

#### 1）导入pom

```xml
 <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>
```

#### 2）加上@EnableDubbo注解 

```java
@EnableDubbo
@SpringBootApplication
public class BootOrderServiceConsumerApplication {

   public static void main(String[] args) {
      SpringApplication.run(BootOrderServiceConsumerApplication.class, args);
   }

}
```

#### 3）修改application.yml

​	消费者

```yml
dubbo:
  application:
    name: boot-order-service-consumer
  registry:
    address: zookeeper://192.168.214.30:2181
  monitor:
    address: 192.168.214.30:7070
```

​	提供者

```java
dubbo:
  application:
    name: boot-user-service-provider
  registry:
    address: zookeeper://192.168.214.30:2181
  protocol:
    name: dubbo
    port: 20880
```

​	最主要的不同是不配置暴露服务和实现类，用注解代替

```java
<!--对外暴露的接口 -->
<dubbo:service interface="com.wei.dubbo.service.UserService" ref="userService"></dubbo:service>
<!--处理的实现类-->
<bean name="userService" class="com.wei.dubbo.service.impl.UserServiceImpl"></bean>
```
​	提供者使用@Service注解暴露服务

```java
@Service//是dubbo下的注解
@Component
public class UserServiceImpl implements UserService {

    @Override
    public List<User> getUserList(){
        List<User> list = new ArrayList<>();
        User user1 = new User("weiwei", "wwww");
        User user2 = new User("jaiqi", "xxxx");
        list.add(user1);
        list.add(user2);
        return list;
    }
}
```

​	消费者使用@Reference来导入服务的代理对象

```java
@Service//spring的注解
public class OrderServiceImpl implements OrderService{

	//dubbo会注入远程服务的代理对象
    @Reference
    private UserService userService;

    @Override
    public List<User> getUserList(){
       return userService.getUserList();
    }
}
```

### 2、使用原版的配置文件

#### 1） 导入pom

```xml
 <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>
```

#### 2）启动类添加@ImportResource注解

```java
@ImportResource(locations = "classpath:provider.xml")
@SpringBootApplication
public class BootUserServiceProviderApplication {

   public static void main(String[] args) {
      SpringApplication.run(BootUserServiceProviderApplication.class, args);
   }

}
```

#### 3）添加配置文件 provider.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">


    <context:component-scan base-package="com.wei.dubbo.*"></context:component-scan>
    <dubbo:application name="user-service-provider" ></dubbo:application>

    <dubbo:registry address="zookeeper://192.168.214.30:2181"></dubbo:registry>

    <dubbo:protocol port="20880" name="dubbo"></dubbo:protocol>

    <dubbo:monitor address="192.168.214.30:7070"></dubbo:monitor>

    <!--对外暴露的接口 -->
    <dubbo:service interface="com.wei.dubbo.service.UserService" ref="userService"></dubbo:service>
    <!--处理的实现类-->
    <bean name="userService" class="com.wei.dubbo.service.impl.UserServiceImpl"></bean>
</beans>
```

### 3、使用配置类的方式

#### 1）写dubbo的配置类

```java
package com.wei.dubbo.service.config;

import com.alibaba.dubbo.config.*;
import com.wei.dubbo.service.UserService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author 为为
 * @create 7/27
 */
@Configuration
public class MyDubboConfig {

    @Bean
    public ApplicationConfig applicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("user-service-provider");
        return applicationConfig;
    }

    @Bean
    public RegistryConfig registryConfig() {
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setAddress("zookeeper://192.168.214.30:2181");
        return registryConfig;
    }

    @Bean
    public ProtocolConfig protocolConfig() {
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setPort(20880);
        protocolConfig.setName("dubbo");
        return protocolConfig;
    }

    @Bean
    public MonitorConfig monitorConfig() {
        MonitorConfig monitorConfig = new MonitorConfig();
        monitorConfig.setAddress("192.168.214.30:7070");
        return monitorConfig;
    }

    @Bean
    public ServiceConfig<UserService> serviceConfig(UserService userService) {
        ServiceConfig<UserService> objectServiceConfig = new ServiceConfig<>();
        objectServiceConfig.setInterface(UserService.class);
        objectServiceConfig.setRef(userService);
        return objectServiceConfig;
    }
}
```

#### 2）启动类配置包扫描规则

```java
@EnableDubbo(scanBasePackages = "com.wei.dubbo")
@SpringBootApplication
public class BootUserServiceProviderApplication {

   public static void main(String[] args) {
      SpringApplication.run(BootUserServiceProviderApplication.class, args);
   }

}
```

3）使用dubbo的@Service注解暴露服务

```java
@Service
@Component
public class UserServiceImpl implements UserService {

    @Override
    public List<User> getUserList(){
        List<User> list = new ArrayList<>();
        User user1 = new User("weiwei", "wwww");
        User user2 = new User("jaiqi", "xxxx");
        list.add(user1);
        list.add(user2);
        return list;
    }
}
```