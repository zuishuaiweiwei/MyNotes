# Spring Cloud随笔

## 初步认识

​	cloud是许多技术和思想的结合，从创建步骤来看，使用了maven聚合的特性，以前一个项目巨大无比的情况下，可以分出许多的module，单独拆出功能模块，又可以相互调用。核心思想就是以前单独的一个java进程现在分成许多的微服务，比如原来的一个电商项目，里面有商品模块，订单模块，支付模块等，都在一个包里，以微服务的思想起码要10几个module，所有的微服务一起运行，所有这个缺点很明显，占用内存大，互相调用微服务消耗资源。

​	cloud是分布式开发的一站式解决方案，比较重要的有eureka、ribbon、Hystrix，feign等

## Rest微服务

### provider

​	在创建许多的module模块中，其中有provider，provider的RequestMapper路径要是Rest风格，都返回json格式数据。这样有助于前后端分离，前端工程师只需要指定数据的json格式，后端工程师提供微服务访问地址即可。不同的微服务直接不耦合，可以单独进行开发。

```java
@RequestMapping(value = "user/add",method = RequestMethod.POST)
    public boolean add(@RequestBody User user){
        return userService.add(user);
    }
    @RequestMapping(value = "user/update", method = RequestMethod.POST)
    public boolean update(@RequestBody User user){
        return userService.update(user);
    }
    @RequestMapping(value = "user/list",method = RequestMethod.GET)
    public List<User> list(){
        return userService.list();
    }
    @RequestMapping(value = "user/delete",method = RequestMethod.POST)
    public boolean delete(@RequestBody String id){
        return userService.delete(id);
    }

    @RequestMapping(value = "user/get/{id}",method = RequestMethod.GET)
    public User get(@PathVariable("id") String id){
        return userService.get(id);
    }

```

### consumer

​	微服务的消费者，在contorller中，直接注入RestTemplate使用http访问微服务的提供者。

```java
 @Autowired
    RestTemplate restTemplate;

    private static final String url = "http://provider-user";

@RequestMapping(value = "consumer/user/add", method = RequestMethod.POST)
    public boolean add(@RequestBody User user) {
        return restTemplate.postForObject(url,user,Boolean.class);
    }

    @RequestMapping(value = "consumer/user/update", method = RequestMethod.POST)
    public boolean update(@RequestBody User user) {
        return restTemplate.postForObject(url,user,Boolean.class);
    }

    @RequestMapping(value = "consumer/user/list", method = RequestMethod.GET)
    public List<User> list() {
        return restTemplate.getForObject(url+"user/list",List.class);
    }

    @RequestMapping(value = "consumer/user/delete", method = RequestMethod.POST)
    public boolean delete(@RequestBody String id) {
        return restTemplate.postForObject(url,id,Boolean.class);
    }

    @RequestMapping(value = "consumer/user/get/{id}", method = RequestMethod.GET)
    public User get(@PathVariable("id") String id) {
        return restTemplate.getForObject(url+"user/get/"+id,User.class);
    }
```

### api

​	是所有module的公共模块，放实体类

## Eureka

​	**服务注册于发现**

​	类似一个调度站，用户访问一个服务，先查看有哪些服务提供者可以提供服务，再由eureka进行选择哪个服务提供者

### server

​	

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

```yml

server:
  port: 7001  #服务端口号
eureka:
  instance:
    hostname: eureka7001.com   #实例名称
  client:
    register-with-eureka: false  #表示自己不注册成为一个服务
    fetch-registry: false   #表示自己就是注册中心，不需要去检索服务
    service-url:
     defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/

```



### client

​	客户端就是微服务的provider，需要在eureka注册

```xml
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starts-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

```

```yml

server:
  port: 8001

mybatis:
  type-aliases-package: com.wei.entity
  mapper-locations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/springcloud?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8&allowMultiQueries=true
    username : root
    password : root
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    filters : stat
    maxActive : 1000
    initialSize : 100
    maxWait : 60000
    minIdle : 500
    timeBetweenEvictionRunsMillis : 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: select 1
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    maxOpenPreparedStatements: 20
  application:
    name: provider-user
eureka:
  client:
    service-url:
     defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    instance-id: provider-user8001
    prefer-ip-address: true
info:
  app:
    name: ${spring.application.name}
  build:
    artifactId: ${project.artifactId}
    version: ${project.version}
```

​	启动类上加上@EnableEurekaClient

## Ribbon

​	ribbon是客户端的负载均衡

​	对外暴露的url加入eureka的管理，restTemplate访问provider统一的微服务名称，eureka查找这个微服务名称由哪些提供者，由指定的负载均衡策略确定访问那个微服务提供者

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starts-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
        
         <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

```java
@EnableEurekaClient
@SpringBootApplication
public class ConsumerApp {
```

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate(){
     return new RestTemplate();
 }
```

​	**自定义策略**

​	规则配置类不能和主启动类在一个包下，

![1561138213405](C:\Users\Administrator\Desktop\MyNotes\src\Spring Cloud\img\1561138152351.png)

主启动类加上注解

```java
@RibbonClient(name = "provider" configuration=RibbonConfig.class)
```

​	使用的关键点：restTemplate上加@LoadBalanced注解 表示开启负载均衡，自定义包放的位置，启动类上注解

## Feign

​	feign和ribbon的不同

​		feign是基于接口的开发，ribbon的基于微服务的访问

​		feign集成了ribbon

​		ribbon是consumer使用restTemplate访问微服务名称，api只提供了实体类；feign是consumer使用api提供的新接口加上注解来访问provider

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

​	api的接口

```java
@FeignClient(value = "provider-user")
public interface UserService {

    @RequestMapping(value = "user/add",method = RequestMethod.POST)
    boolean addUser(User user);
    @RequestMapping(value = "user/update", method = RequestMethod.POST)
    boolean updateUser(User user);
    @RequestMapping(value = "user/delete",method = RequestMethod.POST)
    boolean delUserById(String id);
    @RequestMapping(value = "user/list",method = RequestMethod.GET)
    List<User> getList();
    @RequestMapping(value = "user/get/{id}",method = RequestMethod.GET)
    User getUserById(@PathVariable("id") String id);
    @RequestMapping(value = "user/discovery",method = RequestMethod.GET)
    Object discovery();
}
```

​	consumer使用userService

```java
@RestController
public class ConsumerController {

    @Autowired
    UserService userService;

    @RequestMapping(value = "consumer/user/add", method = RequestMethod.POST)
    public boolean add(@RequestBody User user) {
        return userService.addUser(user);
    }

    @RequestMapping(value = "consumer/user/update", method = RequestMethod.POST)
    public boolean update(@RequestBody User user) {
        return userService.updateUser(user);
    }

    @RequestMapping(value = "consumer/user/list", method = RequestMethod.GET)
    public List<User> list() {
        return userService.getList();
    }

    @RequestMapping(value = "consumer/user/delete", method = RequestMethod.POST)
    public boolean delete(@RequestBody String id) {
        return userService.delUserById(id);
    }

    @RequestMapping(value = "consumer/user/get/{id}", method = RequestMethod.GET)
    public User get(@PathVariable("id") String id) {
        return userService.getUserById(id);
    }
    @RequestMapping(value = "consumer/user/discovery", method = RequestMethod.GET)
    public Object discovery() {
        return userService.discovery();
    }
}
```

​	consumer主启动类加上注解

```java
@EnableFeignClients(basePackages = { "com.wei" })
```

## Hystrix

### 熔断

​	熔断就是程序出现异常的时候可以返回可用结果，有点类似回调异常监听

​	provider启动开启熔断

```
@EnableCircuitBreaker
```

​	controller方法加上注解@HystrixCommand

```java
 @RequestMapping(value = "user/get/{id}",method = RequestMethod.GET)
    @HystrixCommand(fallbackMethod = "hystrix_get")
    public User get(@PathVariable("id") String id){
        User user = userService.get(id);
        if(user == null){
            throw new RuntimeException("没有该ID："+id+"的人");
        }
        return user;
    }

    public User hystrix_get(@PathVariable("id") String id){
        return new User().setId(id).setName("没有该ID："+id+"的人 --hystrix_get");
    }
```

### 服务降级

​	熔断是一个方法的回调，显然如果有多个方法写起来麻烦，服务降级是如果服务停止时，客户端不会抛出异常，可以提示用户服务停止了。熔断是客户端的操作和服务器端没有关系

​	在基于feign的面向接口的编程下，修改api。

​	fallbackFactory指定回调类

```java
@FeignClient(value = "provider-user" ,fallbackFactory = UserClientService.class)
public interface UserService {
    @RequestMapping(value = "user/add",method = RequestMethod.POST)
    boolean addUser(User user);
    @RequestMapping(value = "user/update", method = RequestMethod.POST)
    boolean updateUser(User user);
    @RequestMapping(value = "user/delete",method = RequestMethod.POST)
    boolean delUserById(String id);
    @RequestMapping(value = "user/list",method = RequestMethod.GET)
    List<User> getList();
    @RequestMapping(value = "user/get/{id}",method = RequestMethod.GET)
    User getUserById(@PathVariable("id") String id);
    @RequestMapping(value = "user/discovery",method = RequestMethod.GET)
    Object discovery();
}
```

​	服务降级时返回的数据

```java
@Component
public class UserClientService implements FallbackFactory<UserService> {

    @Override
    public UserService create(Throwable throwable) {
        return new UserService() {
            @Override
            public boolean addUser(User user) {
                return false;
            }

            @Override
            public boolean updateUser(User user) {
                return false;
            }

            @Override
            public boolean delUserById(String id) {
                return false;
            }

            @Override
            public List<User> getList() {
                User user = new User().setName("服务降级");
                List list = new ArrayList();
                list.add(user);
                return list;
            }

            @Override
            public User getUserById(String id) {
                return new User().setName("服务降级");
            }

            @Override
            public Object discovery() {
                return null;
            }
        };
    }
}
```

​	consumer修改yml

```yml
feign:
  hystrix:
    enabled: true
```

### DashBoard

​	创建监听module

​	添加pom

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-hystrix-dashboard -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
            <version>2.1.2.RELEASE</version>
        </dependency>
<!-- 可能会不能使用@EnableHystrixDashboard注解 -->
```

​	启动类添加注解

```java
@SpringBootApplication
@EnableHystrixDashboard
public class Hystrix_DashBoard {

    public static void main(String [] args){
        SpringApplication.run(Hystrix_DashBoard.class);
    }
}
```

​	2.0版本下需要监听的服务需要添加一个bean

```java
   @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
```

