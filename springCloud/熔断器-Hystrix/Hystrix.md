## Hystrix:用于隔离远程服务，防止出现雪崩

#### 1、Hystrix降级：当服务器发生异常或调用超时，返回默认数据

##### 1-1、服务端降级

**第一步**：在服务端(provide)引入依赖

```xml
    <dependencies>
        <!--spring boot web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!-- eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
       <!--  hystrix   -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
    </dependencies>
```

**第二步：**在启动类添加注解@EnableCircuitBreaker 开启熔断器

**第三步:**在controller层添加降级方法

当出现异常时，调用降级方法

```java
@GetMapping("/findOne/{id}")
    @HystrixCommand(fallbackMethod = "findOne_fallBack")
    public Goods findOne(@PathVariable("id") int id){
        if (id==1){
            int i=1/0;
        }
        Goods goods = goodsService.findOne(id);
        goods.setTitle(goods.getTitle() + ":" + port);//将端口号，设置到了 商品标题上
        return goods;
    }
//    降级方法
    public Goods findOne_fallBack(int id){
        Goods goods = goodsService.findOne(id);
        goods.setTitle("降级了----");
        return goods;
    }
```

**超时时间的配置**：在@HystrixCommand注解里面添加commandProperties属性，commandProperties需要一个@HystrixProperty类型的数组，在类HystrixCommandProperties里面找到相应的属性值

```java
@HystrixCommand(fallbackMethod = "findOne_fallBack",commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "2000")
})
```

##### **1-2、消费端降级**

**由于我们消费端使用的是fegin完成的远程调用，feign内部已经集成了Hystrix组件，feign可以简化Hystrix的使用，不需要引入Hystrix的依赖**

**第一步：**开启feign对hystrix的支持

```yml
#开启feign对Hystrix的支持
feign:
  hystrix:
    enabled: true
```

**第二步：**重写添加了注解@FeignClient的接口

```java
package com.itheima.consumer.feign;

import com.itheima.consumer.domain.Goods;
import org.springframework.stereotype.Component;

@Component // 将FeignClient的实现类加入到IOC容器
public class GoodsFeignClientFallback implements GoodsFeignClient {
    @Override
    public Goods findGoodsById(int id) {
        Goods goods = new Goods();
        goods.setTitle("又降级了");
        return goods;
    }
}
```

**第三步：**在注解@FeignClient里面添加属性fallback,fallback的值为：重写添加了注解@FeignClient的接口的实现类

```java
@FeignClient(value = "HYSTRIX-PROVIDER",fallback = GoodsFeignClientFallback.class)
```





***不论是在服务端还是消费端都可以触发降级，如果服务端睡眠2s,会触发消费端的降级，原因是消费端的默认超时时间为1s。如果服务端出现异常，会触发服务端的降级。***

------------------------------------------------------------------------------------------------------------------------------------------------------

##### 1-3、熔断器

**Hystrix熔断机制用于对服务的监控，当失败的情况到达预定的阈值(五秒钟内失败20次/降级次数太多),会打开断路器阻止一切请求，值到恢复正常为止。**

**触发断路器参数设置：去HystrixCommandProperties类查找参数属性名**

```java
 @HystrixCommand(fallbackMethod = "findOne_fallBack",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "2000"),
//          失败次数:默认20次
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),
//          监控时间:默认五秒
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "2000"),
//          失败率:默认百分之50
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "50")
    })
```

##### **1-4、熔断监控**

**聚合监控Turbine**

**第一步：**搭建监控模块

**1. 创建监控模块**

创建hystrix-monitor模块，使用Turbine聚合监控多个Hystrix dashboard功能,

**2. 引入Turbine聚合监控起步依赖**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>hystrix-parent</artifactId>
        <groupId>com.itheima</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>hystrix-monitor</artifactId>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

**3. 修改application.yml**

```yml
spring:
  application.name: hystrix-monitor
server:
  port: 8769
turbine:
  combine-host-port: true
  # 配置需要监控的服务名称列表
  app-config: hystrix-provider,hystrix-consumer
  cluster-name-expression: "'default'"
  aggregator:
    cluster-config: default
  #instanceUrlSuffix: /actuator/hystrix.stream
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

```

**4. 创建启动类**

```java
@SpringBootApplication
@EnableEurekaClient

@EnableTurbine //开启Turbine 很聚合监控功能
@EnableHystrixDashboard //开启Hystrix仪表盘监控功能
public class HystrixMonitorApp {

    public static void main(String[] args) {
        SpringApplication.run(HystrixMonitorApp.class, args);
    }

}

```

**第二步：**修改被监控模块

需要分别修改 hystrix-provider 和 hystrix-consumer 模块：

**1、导入依赖：**

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
```

**2、配置Bean**

此处为了方便，将其配置在启动类中。

```java
@Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/actuator/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
```

**3、启动类上添加注解@EnableHystrixDashboard**

```java
@EnableDiscoveryClient
@EnableEurekaClient
@SpringBootApplication
@EnableFeignClients 
@EnableHystrixDashboard // 开启Hystrix仪表盘监控功能
public class ConsumerApp {


    public static void main(String[] args) {
        SpringApplication.run(ConsumerApp.class,args);
    }
    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/actuator/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}

```

**第三步：**启动测试

**1、启动服务：**

- eureka-server

- hystrix-provider

- hystrix-consumer

- hystrix-monitor

**2、访问：**

在浏览器访问http://localhost:8769/hystrix/ （监控服务的端口）进入Hystrix Dashboard界面

![1585421193757](E:/java/JavaEE在线就业班2.0加密/配套资料/5.阶段五-流行框架/Spring Cloud 第二天/资料/1. turbine 搭建/1585421193757.png)

界面中输入监控的Url地址 http://localhost:8769/turbine.stream，监控时间间隔2000毫秒和title，如下图

![1585421278837](E:/java/JavaEE在线就业班2.0加密/配套资料/5.阶段五-流行框架/Spring Cloud 第二天/资料/1. turbine 搭建/1585421278837.png)



- 实心圆：它有颜色和大小之分，分别代表实例的监控程度和流量大小。如上图所示，它的健康度从绿色、黄色、橙色、红色递减。通过该实心圆的展示，我们就可以在大量的实例中快速的发现故障实例和高压力实例。
- 曲线：用来记录 2 分钟内流量的相对变化，我们可以通过它来观察到流量的上升和下降趋势。

![1585421278837](E:/java/JavaEE在线就业班2.0加密/配套资料/5.阶段五-流行框架/Spring Cloud 第二天/资料/1. turbine 搭建/1167856120180.png)