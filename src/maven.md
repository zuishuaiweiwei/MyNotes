### jstl 

```xml
 <dependency>
					            <groupId>jstl</groupId>
					            <artifactId>jstl</artifactId>
					            <version>1.2</version>
					            <exclusions>
					                <exclusion>
					                    <groupId>javax.servlet</groupId>
					                    <artifactId>servlet-api</artifactId>
					                </exclusion>
					                <exclusion>
					                    <groupId>javax.servlet.jsp</groupId>
					                    <artifactId>jsp-api</artifactId>
					                </exclusion>
					            </exclusions>
					        </dependency>
					        <!-- taglib -->
					        <dependency>
					            <groupId>taglibs</groupId>
					            <artifactId>standard</artifactId>
					            <version>1.1.2</version>
					            <exclusions>
					                <exclusion>
					                    <groupId>javax.servlet</groupId>
					                    <artifactId>servlet-api</artifactId>
					                </exclusion>
					                <exclusion>
					                    <groupId>javax.servlet.jsp</groupId>
					                    <artifactId>jsp-api</artifactId>
					                </exclusion>
					            </exclusions>
					        </dependency>
```

### servlet依赖

```xml
<dependency>
						            <groupId>javax.servlet</groupId>
						            <artifactId>servlet-api</artifactId>
						            <version>2.5</version>
						            <scope>provided</scope>
						</dependency>
						<dependency>
						            <groupId>javax.servlet.jsp</groupId>
						            <artifactId>jsp-api</artifactId>
						            <version>2.1</version>
						            <scope>provided</scope>
						</dependency>
```

### druid

```xml
<dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.0.9</version>
            </dependency>
```

### oracle

```xml
<dependency>
							<groupId>mysql</groupId>
							<artifactId>mysql-connector-java</artifactId>
							<scope>runtime</scope>
						</dependency>	
```

### mysql

```xml
<dependency>
							<groupId>mysql</groupId>
							<artifactId>mysql-connector-java</artifactId>
							<scope>runtime</scope>
						</dependency>
```

### mybatis（单独）

```xml
<dependency>    
							<groupId>org.mybatis</groupId>    
							<artifactId>mybatis</artifactId>    
							<version>3.3.0</version>    
						</dependency>    
				<!-- mybatis-spring-->   
						<dependency>    
							<groupId>org.mybatis</groupId>    
							<artifactId>mybatis-spring</artifactId>    
							<version>1.2.3</version> 
						</dependency>  
```

### spring-boot

```xml

						<dependency>
							<groupId>org.springframework.boot</groupId>
							<artifactId>spring-boot-starter-web</artifactId>
						</dependency>
		<!-- thymeleaf -->
						<dependency>
							<groupId>org.springframework.boot</groupId>
							<artifactId>spring-boot-starter-thymeleaf</artifactId>
						</dependency>

		<!-- redis -->		
					<dependency>
						<groupId>org.springframework.boot</groupId>
						<artifactId>spring-boot-starter-data-redis</artifactId>
					</dependency>
		<!-- jsr303 -->				
					  <dependency>
						<groupId>org.mybatis.spring.boot</groupId>
						<artifactId>spring-boot-starter-validation</artifactId>
					</dependency>
		<!-- mybatis -->	
					  <dependency>
						  <groupId>org.mybatis.spring.boot</groupId>
						  <artifactId>mybatis-spring-boot-starter</artifactId>
						  <version>1.3.1</version>
					  </dependency>
```

### springboot热部署

```xml
<dependency>
						<groupId>org.springframework.boot</groupId>
						<artifactId>spring-boot-devtools</artifactId>
						<optional>true</optional>
						<scope>true</scope>
					</dependency>
```

### eureka

```xml
服务端
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

```xml
客户端
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



### md5 

```xml
<dependency>
						<groupId>commons-codec</groupId>
						<artifactId>commons-codec</artifactId>
						<version>1.9</version>
					</dependency>	
```

### ueditor

```xml
<dependency>
						<groupId>com.baidu</groupId>
						<artifactId>ueditor</artifactId>
						<version>1.1.2</version>
					</dependency>
					<dependency>
						<groupId>org.json</groupId>
						<artifactId>json</artifactId>
					</dependency>
					<dependency>
						<groupId>com.baidu.ueditor</groupId>
						<artifactId>json</artifactId>
						<version>1.0</version>
					</dependency>

					<dependency>
						<groupId>commons-io</groupId>
						<artifactId>commons-io</artifactId>
						<version>2.4</version>
					</dependency>
```

## Shiro

### html+shiro

```java
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>

    <html lang="en" xmlns:th="http://www.thymeleaf.org"
          xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
          
 @Bean
 public ShiroDialect shiroDialect() {
    return new ShiroDialect();
 }

```

### ehcache

```xml
<dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>1.3.2</version>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-api</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```

## Redis

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency> 
```

