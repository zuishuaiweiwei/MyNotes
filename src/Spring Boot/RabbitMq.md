# RabbitMq在SpringBoot中的使用

## 1、简单测试

### 1、首先简单配置yml文件

![](/images/QQ截图20190423160725.png)



### 2、编写发送者

​	

```java


@Test
	public void sender() {
        User user = new User();
        user.setName("weiwei");
        user.setPwd("wei");
        /*HashMap<Object, Object> map = new HashMap<>();
        map.put("user","weiwei");
        map.put("list", Arrays.asList("a","b"));*/
        rabbitTemplate.convertAndSend(/*发送给那个交换器*/"exchange.direct",/*发送的路由键*/"wei.user",/*发送的内容*/user);
	}
```



### 3、接收者

```java
	@Test
    public void reciver(){
        //Message receive = rabbitTemplate.receive("wei.user");
        //receive.getBody(); //取内容
        //receive.getMessageProperties();  //获取一些头文件
        Object receive = rabbitTemplate.receiveAndConvert("wei.user");
        System.out.println(receive.getClass());
        System.out.println(receive);
    }
```



### 4、配置MessageConverter



​	springboot默认使用SimpleMessageConverter里面默认使用jdk的序列化，需要自己配置转化器。

```java
@Component
public class MyMessageConverter {
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }

}
```

## 2、项目中使用

需要配合使用的注解

### @EnableRabbit

​	写在主类上，开启rabbitmq的注解模式

```java
@EnableRabbit
@SpringBootApplication
public class AmqpApplication {

	public static void main(String[] args) {
		SpringApplication.run(AmqpApplication.class, args);
	}

}
```



### @RabbitListener

​	写在方法上用来监听队列

```java
@Service
public class UserService {

    @RabbitListener(queues = "wei.user")
    public void receiver(User user){
        System.out.println("接受到了数据");
        System.out.println(user);
    }
    
}
```

​	主要属性

|       |                                    |
| ----- | ---------------------------------- |
| queue | 表示监听的队列，可以放数组监听多个 |

## 3、使用AmqpAdmin创建exchange，queue

​	

```java
    @Autowired
    AmqpAdmin amqpAdmin;
    @Test
    public void admin(){
        //创建exchange
        amqpAdmin.declareExchange(new DirectExchange("amqp.direct"));
        //删除queue
        amqpAdmin.deleteQueue("amqp.direct");
        //创建queue
        amqpAdmin.declareQueue(new Queue("amqp.queue",true));
        //绑定queue
        amqpAdmin.declareBinding(new Binding(/*要绑定的queue*/"amqp.queue", 
                /*绑定类型*/Binding.DestinationType.QUEUE,/*绑定的交换器*/"amqp.direct",/*路				 由规则*/"amqp.test",/*头信息*/null));
    }
```

