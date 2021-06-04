**springCloud Bus是一个构建消息驱动服务的应用框架，也就是用同一套代码使用不同的消息中间件，可惜stream只能实现rabbitmq与kaffka的切换。使得开发人员可以无感知的使用消息中间件，使得微服务的开发高度解耦，更多的去关注自己的业务。类似于jdbc为各类数据库提供接口，把各类数据库的实现细节屏蔽掉，应用程序只需要对jdbc进行编程即可 **。

![image-20210410170655841](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210410170655841.png)

### 消息生产者

**第一步：创建消息生产者模块**

**第二步：引入依赖**

```xml
<dependencies>
    <!--spring boot web-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- stream -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    </dependency>
</dependencies>
```

**第三步：配置application.yml文件**

```yml
spring:
  cloud:
    stream:
      binders: #定义绑定器
        itheima_binder: #自定义绑定名称
          type: rabbit #绑定器类型
          environment: #rabbitmq的环境
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                virtual-host: /
                username: guest
                password: guest
      bindings:
        output: #管道名称
          binder: itheima_binder #指定使用哪一个binder
          destination: itheima_exchange #消息目的地交换机


server:
  port: 8000
```

**第四步：创建一个消息生产者对象注入的MessageChannel对象的名称不要乱取使用output，否则会报错**

```java
package com.hdf.produce;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.context.annotation.Primary;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;
import org.springframework.messaging.MessageChannel;
@Component
@EnableBinding(Source.class)
public class MessageProducer {
    @Autowired
    private MessageChannel output;
    public void sendMessage(){
        String m="hello stream";
        output.send(MessageBuilder.withPayload(m).build());
        System.out.println("消息发送成功！！！");
    }
}
```

### 消息消费者

**第一步：创建消费者模块**

**第二步：引入依赖**

```xml
<dependencies>
    <!--spring boot web-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- stream -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    </dependency>
</dependencies>
```

**第三步：配置application.yml文件**

```yml
spring:
  cloud:
    stream:
      binders: #定义绑定器
        itheima_binder: #自定义绑定名称
          type: rabbit #绑定器类型
          environment: #rabbitmq的环境
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                virtual-host: /
                username: guest
                password: guest
      bindings:
        output: #管道名称
          binder: itheima_binder #指定使用哪一个binder
          destination: itheima_exchange #消息目的地交换机


server:
  port: 9000
```

**第四步：创建一个消息消费者的类**

```java
package com.hdf.consumer;

import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;

@Component
@EnableBinding(Sink.class)
public class MessageListener {
    @StreamListener(Sink.INPUT) //接收消息需要一个管道 与配置文件中管道名称一致
    public void receive(Message message){
        System.out.println(message);
        System.out.println(message.getPayload());
    }
}
```