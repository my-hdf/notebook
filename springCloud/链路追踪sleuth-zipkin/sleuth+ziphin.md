**sleuth进行数据的收集，zipkin进行数据的图形化界面展示**

**第一步：启动zipkin**

访问：http://localhost:9411/

**第二步：引入zipkin的依赖**

因为zipkin内部依赖了sleuth所有不需要引入

```xml
<!-- zipkin -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

**第三步：在consumer和produce的配置文件中添加配置**

```yml
spring:
  zipkin:
    base-url: http://localhost:9411/ #zipkin的服务地址
  sleuth:
    sampler:
      probability: 1
```