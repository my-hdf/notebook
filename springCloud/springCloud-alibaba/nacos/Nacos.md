起步

添加依赖

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
```

添加配置信息

```properties
server.port=8081 #端口号
spring.application.name=nacos-producer #应用名称
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848 #地址
management.endpoints.web.exposure.include=*
```

