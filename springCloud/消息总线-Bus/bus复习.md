**什么是消息总线？**

我的理解是通过轻量级的消息代理(rabbitmq、kafka)构建一个共同的消息组题，所有微服务都与之连接，从而微服务实例可以对其进行监控与消费，这就是消息总线。

**消息总线如何实现？**

消息总线的实现需要依赖于消息中间件

服务端：

-------------------------------------------------------------------------------------------

导入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
 </dependency>
```

暴露端点 用于刷新

```yml
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```

客户端：

-----------------------------------------------------------------------------------------------------------------------------------

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
 </dependency>
```

暴露端点

```yml
management:
  endpoints:
    web:
      exposure:
        include: '*'  #暴露所有端点便于监控
```

在需要进行刷新数据的地方添加@RefreshScope注解

![image-20210602151500135](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602151500135.png)

--------------------------------------------------------------------------------------------------------

通过刷新总控(3700)来实现消息的更新

**原理图**

![image-20210602153451930](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602153451930.png)

**curl -X POST http://localhost:3700/actuator/bus-refresh**

![image-20210602152344432](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602152344432.png)

定点刷新(让需要进行配置更新的微服务进行更新)

**curl -X POST http://localhost:3700/actuator/bus-refresh/微服务：端口号**

![image-20210602153112137](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602153112137.png)

效果

总控3700

![image-20210602153156155](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602153156155.png)

客户端3701

![](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602153232570.png)

客户端3702

![image-20210602153258829](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602153258829.png)

