### 1、降级处理在服务的提供方

当服务的提供方处理超时时，使用hystrix的降级处理

加上@HystrixCommand注解

```java
@HystrixCommand(fallbackMethod ="toPay_fallback",commandProperties = {
        @HystrixProperty(name ="execution.isolation.thread.timeoutInMilliseconds",value = "2000")
})
public String toPay(Integer id) {
        try {
            TimeUnit.SECONDS.sleep(5);
        }catch (Exception e){
            e.printStackTrace();
        }
        //int i=1/0;
        return "当前线程是: "+Thread.currentThread().getName()+"  id"+id;
    }
```

添加降级方法

```java
public String toPay_fallback(Integer id){
    return "当前线程是: "+Thread.currentThread().getName()+"id:"+id+" 请求超时，请稍后重试";
}
```

开启断路器

```java
@EnableCircuitBreaker  //开启断路器
```

 @HystrixProperty就像是一根警戒线，当处理时间超过指定的时间时，开启降级处理，调用toPay_fallback方法

在测试过程中发现当代码出现异常时，同样会开启降级处理int i=1/0

### 2、降级处理方法在消费端

当服务的消费方请求超时或出现异常时，启用服务降级

修改yml配置文件 添加配置 

```yml
feign:
  hystrix:
    enabled: true
```

开启hystrix

```java
@EnableHystrix
```

添加降级方法

​       @DefaultProperties(defaultFallback = "getMessageError")

​        默认降级方法，当没有具体指定降级方法时，使用默认降级方法，使用默认降级，解决了代码膨胀的问题

​       

```java
package com.hdf.hystrix.controller;

import com.hdf.hystrix.feign.HystrixProviderFeign;
import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/hystrix")
//默认降级方法，当没有具体指定降级方法时，使用默认降级方法
@DefaultProperties(defaultFallback = "getMessageError")
public class HystrixConsumerController {
    @Autowired
    private HystrixProviderFeign hystrixProviderFeign;
    @RequestMapping("/getOrder/{id}")
    @HystrixCommand(fallbackMethod = "getOrderTimeout",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "2000")
    })
    public String getOrder(@PathVariable("id") Integer id){
        String s = hystrixProviderFeign.toPay(id);
        return s;
    }
    public String getOrderTimeout(Integer id){
        return "ThreadName:"+Thread.currentThread().getName()+" id:"+id+"  生成订单请求超时！！！稍后重试";
    }
    @RequestMapping("/message/{id}")
    @HystrixCommand
    public String getMessage(@PathVariable("id") Integer id){
        int i=1/0;
        return "ThreadName:"+Thread.currentThread().getName()+" id:"+id;
    }
    public String getMessageError(){
        return "ThreadName:"+Thread.currentThread().getName()+"请求异常，请稍后重试";
    }
}
```

将降级方法与feign接口绑定，解决代码混乱的问题（业务逻辑代码和降级方法混乱）。避免了每一个业务逻辑方法下面都需要配一个降级方法。

创建一个类实现feign接口

```java
package com.hdf.hystrix.feign.Feignfallback;

import com.hdf.hystrix.feign.HystrixProviderFeign;
import org.springframework.stereotype.Component;

@Component
public class MyHystrixProviderFeignFallback implements HystrixProviderFeign {
    @Override
    public String toPay(Integer id) {
        return "服务器异常请稍后重试！！！";
    }
}
```

在feign接口的@FeignClient(value = "HYSTRIX-PROVIDER8011",fallback = MyHystrixProviderFeignFallback.class)里面添加一个fallback属性，属性值为feign接口的实现类

当调用feign接口出现异常、超时、服务提供方宕机时，会调用feign接口实现方法的降级方法

**在学习中发现使用hystrix后，ribbon的readTimeout、connectTimeout属性失去作用，这时我们需要添加以下配置**

```yml
ribbon:
  #  连接之后，读取数据的超时时间
  ReadTimeout: 5000
  #  连接超时时间
  ConnectTimeout: 5000  #失去作用
#添加下面配置
hystrix:
  command:
    default:
      execution:
        timeout:
          enabled: false
 feign:
  hystrix:
    enabled: true
```

### 3、服务熔断

<img src="C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210530202018758.png" alt="image-20210530202018758" style="zoom:67%;" />

​       当请求次数达到一定的阀值(10秒请求20次)，并且失败率达到50%(10秒内20次请求失败10次以上)时，断路器打开。这时请求到达时(无论是正常请求还是错误请求)都会使用降级方法充当主逻辑方法。大约在5秒后，熔断器会处于一个半开状态，允许一个请求通过，当请求访问正常，则关闭熔断器，否在重复之前操作。

熔断器配置相关类HystrixCommandProperties

![image-20210530203009202](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210530203009202.png)

### 4、服务限流

。。。。。。。。。。。。。。。。。。。。。。。

### 5、服务监控

1、创建一个监控微服务

导入坐标

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

...............

启动微服务进行监控

http：//localhost:port/hystrix

2、监控微服务

1、需要监控的微服务导入坐标

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2、添加配置

```yml
#暴露所有端点
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

3、在启动类上添加一个bean

```java
@Bean
public ServletRegistrationBean getServlet(){
    HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
}
```

进入http：//localhost:port/hystrix，在地址栏中输入http://localhost:port/hystrix.stream

![image-20210530214915340](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210530214915340.png)

如果出现以下情况

![image-20210530215003013](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210530215003013.png)

看看控制台日志

![image-20210530215041424](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210530215041424.png)

在监控微服务里面添加配置

```yml
hystrix:
    dashboard:
    proxy-stream-allow-list: localhost
```

再次访问出现

![image-20210530215333939](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210530215333939.png)

