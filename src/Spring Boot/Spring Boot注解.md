# Spring Boot注解

## 一、cache有关注解

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

## 二、Controller有关注解



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



## 三、RabbitMq有关的注解

### @EnableRabbit

​	写在主类上，开启rabbitmq的注解模式

### @RabbitListener

​	监听队列

​	主要属性

|       |                                    |
| ----- | ---------------------------------- |
| queue | 表示监听的队列，可以放数组监听多个 |



------

## 四、ElasticSearch有关注解



## 五、Async有关注解

### 	@Async

​	标注在需要异步的方法上，需要配合@EnableAsync一起使用

### 	@EnableAsync

​	开启异步

```java
@EnableAsync
@SpringBootApplication
public class AsyncApplication {
    
    
@Service
public class HelloService {

    @Async
    public void hello(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("hello");

    }
}
    
@RestController
public class HelloController {
    @Autowired
    HelloService helloService;

    @GetMapping("hello")
    public String hello(){
        helloService.hello();
        return "success";
    }
}
```

​	加上异步的注解，浏览器会立即显示success 不需要等待3秒

## 六、定时任务有关的注解

### 	@Scheduled

注解在需要定时的方法上,需要配合@EnableScheduling进行使用

### 	@EnableScheduling

开启任务调度

```java
@EnableScheduling
@SpringBootApplication
public class AsyncApplication {

	public static void main(String[] args) {
		SpringApplication.run(AsyncApplication.class, args);
	}

}

@Service
public class ScheduledService {

    @Scheduled(cron = "0 * * * * MON-SAT")
    public void test(){
        System.out.println("hello");
    }
}
```

​		主要属性

| cron | 配置需要什么时间调度任务 |
| ---- | ------------------------ |
|      |                          |

​	有6个位置可以配置（* * * * * *）

| 特殊字符 |                       |
| -------- | --------------------- |
| ，       | 表示枚举              |
| *        | 表示任意              |
| -        | 表示范围              |
| /        | 表示步长              |
| ？       | 星期/日期冲突匹配     |
| L        | 最后                  |
| W        | 工作日                |
| C        | 和calender计算的结果  |
| #        | 星期，4#2 第二个星期4 |

| 位置              | 可以配置的                         |
| ----------------- | ---------------------------------- |
| 第一个 * 表示秒   | 0-59  ，  -    *    /              |
| 第二个 * 表示分   | 0-59  ，  -    *    /              |
| 第三个 * 表示时   | 0-59  ，  -    *    /              |
| 第四个 * 表示天   | 1-31  ，  -    *    /  ？  L  W  C |
| 第五个 * 表示月   | 1-12  ，  -    *    /              |
| 第六个 * 表示周几 | 0-7或SUN-SAT    0,7都表示星期天    |

​	eg：

​	【0 0/5 14,18 * * ?】每天14点和18点整，每隔5分钟调度一次

​	【0 15 10 ？ * 1-6】每个月周1到周6，10:15 执行一次

​	【0 0 2 ？ * 6L】 每个月的最后一个周6,2:00 执行一次

​	【0 0 2 LW * ?】 每个月的最后一个工作日，2：00执行一次

​	【0 0 2-4 ？ * 1#1】每个月第一个星期1,2点到4点，每个整点都执行一次

​	



