# ElasticSearch在SrpingBoot中的使用

## 一、使用Jest

#### 1、下载pom文件

```java
  		<dependency>
            <groupId>io.searchbox</groupId>
            <artifactId>jest</artifactId>
            <version>6.3.1</version>
        </dependency>
```

#### 2、配置uris地址

​	从JestProperties中知道uris的默认值为localhost：9200 

```yml
spring:
  elasticsearch:
    jest:
      uris: http://192.168.214.30:9200
```

#### 3、编写实体类

```java
public class Book {
    @JestId
    private Integer id;
    private String name;
    private String price;
    private String color;
    private String count;

    public String getName() {
        return name;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPrice() {
        return price;
    }

    public void setPrice(String price) {
        this.price = price;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public String getCount() {
        return count;
    }

    public void setCount(String count) {
        this.count = count;
    }
}
```

#### 4、测试添加数据

```java
 	@Autowired
    JestClient jestClient;

    @Test
	public void contextLoads() {
        Book book = new Book();
        book.setId(1);
        book.setName("红楼梦");
        book.setPrice("50");
        book.setColor("红色");
        book.setCount("30");
        Index index = new Index.Builder(book).index("book").type("books").build();
        try {
            jestClient.execute(index);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

访问 http://192.168.214.30:9200/book/books/1 

```json
{
 "_index":"book",
 "_type":"books",
 "_id":"1",
 "_version":1,
 "_seq_no":0,
 "_primary_term":1,
 "found":true,
 "_source":
 	{"id":1,
     "name":"红楼梦",
     "price":"50",
     "color":"红色",
     "count":"30"
    }
}
```

#### 5、测试获取 数据

```java
@Test
    public void contextLoads2() {
        String query = "{\n" +
                "    \"query\" : {\n" +
                "        \"match\" : {\n" +
                "            \"price\" : \"50\"\n" +
                "        }\n" +
                "    }\n" +
                "}";
        Search search = new Search.Builder(query).addIndex("book").addType("books").build();
        try {
            String string = jestClient.execute(search).getJsonString();
            System.out.println(string);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```

获取到的数据

```java
{"took":67,
"timed_out":false,
"_shards":
	{"total":1,
	"successful":1,
	"skipped":0,
	"failed":0},
	"hits":
		{"total":
			{"value":1,
			"relation":"eq"
			},
			"max_score":0.2876821,
			"hits":[{
                    "_index":"book",
                    "_type":"books",
                    "_id":"1",
                    "_score":0.2876821,
                    "_source":
                            {"id":1,
                            "name":"红楼梦",
                            "price":"50",
                            "color":"红色",
                            "count":"30"}}]}}
```

#### 5、关键点

​	**存储需要构建一个Index，需要内容和指定index（索引）和type（类型）。存的东西必要要有主键ID，可以在实体类中使用@JestId来说明这个是主键。存储的时候就不用指定ID。**

​	**查询数据要构建一个Search，需要指定查询的索引和类型。传递一个查询语句。**

​	**构建出来的Index和Search都是交给JestClient.excute();方法执行；**

------



## 二、使用Spring Data

#### 1、配置pom文件

```java
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
		</dependency>
```

#### 2、配置yml文件

```yml
spring:
  data:
    elasticsearch:
      cluster-name: elasticsearch
      cluster-nodes: 192.168.214.30:9300
```

​	直接启动如果报timeout的错误可能是引文版本不对，spring data elasticsearch版本可能不合适，[版本控制查看](https://github.com/spring-projects/spring-data-elasticsearch)

​	如果报错可以从新下一个合适版本的的elsearch

#### 3、编写