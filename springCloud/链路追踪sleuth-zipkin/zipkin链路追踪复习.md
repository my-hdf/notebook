zipkin是什么？

zipkin用于链路追踪，一个分布式系统往往拥有多个微服务，微服务之间的调用往往形成一条很长的链路。而zipkin(spring cloud sleuth)就是用于追踪定位微服务之间的调用。

zipkin能够干什么？

zipkin用于微服务之间的调用追踪，也可以配合一些压力软件使用，测试微服务系统在大压力下的可用性和性能

zipkin怎么用？

启动zipkin服务器

![image-20210604085024595](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210604085024595.png)

![image-20210604085052779](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210604085052779.png)

给需要进行的微服务进行配置和添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

```yml
management:
  endpoints:
    web:
      exposure:
        include: '*'  #暴露端点用于监控
```

![image-20210604085235566](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210604085235566.png)

