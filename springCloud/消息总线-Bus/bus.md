**Spring Cloud Bus是用轻量级的消息中间件将分布式节点连接起来，可以用于广播配置文件的更改或者服务监控管理。**

### 起步

**第一步：在config-server配置中心导入坐标**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <!-- eureka-client -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    
     <!-- 以下是主要坐标 -->
    <!-- bus -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

**第二步：配置application.xml添加rabbitmq的配置+暴露bus-refresh端点**

```yml
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/he_ding_fa/config_public.git #gittee仓库的地址
      label: master 
  rabbitmq:
    username: guest
    password: guest
    virtual-host: /
    host: localhost
    port: 5672
    
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```

**第三步：在需要获取动态配置的微服务添加坐标和相关配置**

```xml
    <dependencies>

        <!--spring boot web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <!--feign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        
        <!-- config -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>

        <!-- bus -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
    </dependencies>
```

**添加rabbitmq的配置**

```yml
# 配置config-server地址
# 配置获得配置文件的名称等信息
spring:
  cloud:
    config:
      # 配置config-server地址
      # uri: http://localhost:9527
      # 配置获得配置文件的名称等信息
      name: config # 文件名
      profile: dev # profile指定，  config-dev.yml
      label: master # 分支
      # 从注册中心去寻找config-server地址
      discovery:
        enabled: true
        service-id: CONFIG-SERVER
  #主要添加的配置
  #配置rabbitmq信息
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

第四步：动态更新配置信息

打开cmd输入命令curl -X POST http://localhost:9527/actuator/bus-refresh进行对所有与config-server相关联的微服务配置进行更新