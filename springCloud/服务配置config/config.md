**Spring Cloud Config解决了在分布式场景下多环境配置文件的管理和维护**

### 1、起步(config-server远程仓库配置中心创建)

**第一步：创建config-server子模块**

**第二步：导入坐标**

```xml
<dependencies>
    <!-- config-server -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
</dependencies>
```

**第三步：创建引导类并添加注解@EnableConfigServer**

**第四步：创建application.yml文件**

如果是私密仓库需要指定username和password

```yml
server:
  port: 9527
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/he_ding_fa/config_public.git //gitee仓库地址
      label: master //分支
```

### 2、微服务获取远程仓库配置文件数据

**第一步：创建bootstrap.yml配置文件，bootstrap.yml配置文件的优先级高于application.yml**

注意：在远程仓库定义的配置文件命名规则为：文件名-环境.yml

```yml
spring:
  cloud:
    config:
      uri: http://localhost:9527  #config-server的访问uri
      label: master #指定节点
      name: config  #文件名
      profile: dev #环境
```

**第二步：测试**

通过@Value("{参数名}")获取数据并为变量注入

### 3、单节点刷新

**第一步：在需要进行动态刷新的模块添加一个依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**第二步：在获取配置信息的类上加上注解**

@RefreshScope //开启刷新

**第三步：暴露refresh端点**

```yml
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

**第四步：打开cmd对需要刷新的微服务访问路径进行刷新**

curl -X POST http://localhost:8001/actuator/refresh

### **4、config集成Eureka**

1、导入坐标

在config-server模块导入Eureka-client的坐标

2、在config-server模块的application.yml文件添加配置

添加注册到eureka-server的配置

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
```

3、在导入了远程配置文件服务的bootstrap.yml文件添加以下配置

```yml
spring:
  cloud:
    config:
#      uri: http://localhost:9527  #config-server的访问uri
      label: master #指定节点
      name: config  #文件名
      profile: dev #环境
      discovery:
        enabled: true #开启服务发现
        service-id: config-server # 配置中心的应用名称
```