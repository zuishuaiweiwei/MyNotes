# mybatis笔记

## 1.简单使用

​	一般的使用步骤 （以springboot为例）：

​		1）先配置总配置类

```yml
mybatis:
  type-aliases-package: com.wei.entity   # 配置实体类包路径
  mapper-locations: classpath:mapper/*.xml  # 配置mapper文件位置
  configuration:
    map-underscore-to-camel-case: true         #开启驼峰命名
```

​		2）先写实体类和dao接口（**接口使用@Mapper注明是一个mybatis接口类**）

​		3） 配置xxMapper.xml文件

```xml
namespace表示那个用@Mapper注明的接口类
	 id是方法名
 	 resultType是返回值 如果是返回list 也可以用resultType 
	 User是总配置文件中扫描实体类路径下的类名
-->
<mapper namespace="com.wei.dao.UserDao"> 
  
   <select id="getList"  resultType="User">
      SELECT * FROM user
   </select>
    <select id="getUserById"  parameterType="string" resultType="User">
      SELECT * FROM user WHERE id = #{id}
   </select>
</mapper>
```

## 2 返回boolean的问题

```xml
     <delete id="delUserById"  parameterType="string" >
        DELETE FROM user WHERE id = #{id}
    </delete>
    <insert id="addUser" parameterType="User" >
        INSERT INTO user(name,nickname) VALUES (#{name},#{nickName})
    </insert>
    <update id="updateUser" parameterType="User" >
       UPDATE user SET name = #{name},nickName = #{nickName}
        where id = #{id}
    </update>
  <!--  insert update delete都可以返回boolean 比较好的是只要接口里边指定返回boolean，不指定resultType="boolean"也可以正确返回
    0：返回false 1：返回true 
-->
```

