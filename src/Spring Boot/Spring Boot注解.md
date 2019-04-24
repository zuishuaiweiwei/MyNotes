# Spring Boot注解

### 1、 @Cacheable 

可以写在类上和方法上 ，都表示可以缓存的 

主要属性：

| **cacheNames**   | **指定缓存时的的key**                                        |
| ---------------- | ------------------------------------------------------------ |
| **key**          | **相当于同一类对象下的子文件夹 eg:user.a ,user.b a和b是不同的user对象** |
| **condition**    | **判断条件 只有返回true才缓存**                              |
| **cacheManager** | **指定用那个缓存管理器进行缓存**                             |

### 2、@CachePut

作用和cacheeable作用类似，在查询数据的时候直接查询数据库并将数据缓存起来

主要属性参考cacheable

### 3、@CacheConfig

加在类上，为当前类统一缓存配置

缓存格式为  

| **key**   | **CacheConfig的cacheNames 属性：Cacheable中的key属性** |
| --------- | ------------------------------------------------------ |
| **value** | **查询出来的数据**                                     |

主要属性

| **cacheName**    | **生存缓存的key的前缀**  |
| ---------------- | ------------------------ |
| **cacheManager** | **指定使用的缓存管理器** |





### 4、@PathVariable

可以从url上面取值，传递给方法的形参上

```java
@GetMapping("/path/#{id}")
public String test(@PathVariable("id") String id){}
```

### 5、@RequestParam

用来接收URL中的参数,如/path?id=001,可接收001作为参数

```java
@GetMapping("/path")
public String test(@RequestParam("id") String id){}
```

### 6、@RequestAttribute

用于访问由过滤器或拦截器创建的、预先存在的请求属性，效果等同与request.getAttrbute().

```java
@GetMapping("/path")
public String test(@RequestAttribute("id") String id){}
```

### 7、@Component、@Service、@Repository

这三者都是申明一个单例的bean类并纳入spring容器中，后两者其实都是继承于`@Component`。

- @Component 最普通的组件，可以被注入到spring容器进行管理
- @Repository 作用于持久层
- @Service 作用于业务逻辑层
- 

### 8、@RestController

等同于@ResponseBody和@Controller的合集

### 9、@ResponseBody

指定返回的是json数据

### 10、@RequestMapping

负责URL到Controller中的具体函数的映射

```
@RequestMapping("/path")
public String test(@RequestAttribute("id") String id){}

/加不加都一样
```

### 11、@Autowired

注解在成员变量中 ，自动导入依赖的bean

### 12、@Qualifier

当有多个同一类型的Bean时，可以用@Qualifier(“name”)来指定（name是方法的名字）。与@Autowired配合使用。@Qualifier限定描述符除了能根据名字进行注入，但能进行更细粒度的控制如何选择候选者

------



## RabbitMq有关的注解

### @EnableRabbit

​	写在主类上，开启rabbitmq的注解模式

### @RabbitListener

​	监听队列

​	主要属性

|       |                                    |
| ----- | ---------------------------------- |
| queue | 表示监听的队列，可以放数组监听多个 |



------

## ElasticSearch有关注解



