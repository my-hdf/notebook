配置服务中心使用依赖(服务端)

```java
@EnableConfigServer  #配置服务
```

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/my-hdf/myconfig.git #github仓库访问地址
          search-paths:
            - myconfig #查找的文件夹
          username: my-hdf #github用户名
          password: he19980316521 #github密码
          force-pull: true #强制拉取
      label: master #拉取主分支
```

![image-20210601204413930](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210601204413930.png)

配置客户端使用依赖(客户端)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

将application.yml配置文件名改为bootstrap.yml(bootstrap.yml配置文件的优先级高于application.yml的优先级)

```
spring:
  cloud:
    config:
      label: master
      uri: http://localhost:3700  #等价于访问配置微服务http://localhost:3700/master/config-dev.yml
      profile: dev
      name: config
```

在加入依赖和配置文件后，创建一个表现层api,用于访问更新后的数据

```java
@RestController
@RefreshScope //刷新数据
public class ConfigClientController {
    @Value("${config.info}") //获取来自配置服务3700的配置文件数据
    private String configInfo;
    @RequestMapping("/message")
    public String getMessage(){
        return configInfo;
    }
}
```

启动配置客服端服务后访问http://localhost:3701/message

![image-20210601204539494](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210601204539494.png)

> ### `注意：配置客户端(3701)的配置数据是来自于配置服务(3700),而配置服务的数据来自线上github,在3700服务更新完毕数据后，客户端3701不能够及时更新数据，必须要重新启动服务才可以更新数据`

**为了减少重写启动微服务带来的麻烦，我们可以进行手动更新**

首先添加一个actuator监控依赖

在需要进行刷新数据的地方添加一个@RefreshScope注解

```java
@RestController
@RefreshScope //刷新数据
public class ConfigClientController {
    @Value("${config.info}") //获取来自配置服务3700的配置文件数据
    private String configInfo;
    @RequestMapping("/message")
    public String getMessage(){
        return configInfo;
    }
}
```

**cmd输入curl -X POST [http://localhost:3701/actuator/refresh](http://localhost:3701/actuator/refresh)进行手动刷新**

![image-20210601205018949](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210601205018949.png)

