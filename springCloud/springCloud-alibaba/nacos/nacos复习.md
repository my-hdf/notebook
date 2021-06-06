**nacos是什么？**

[https://nacos.io/zh-cn/docs/what-is-nacos.html](https://nacos.io/zh-cn/docs/what-is-nacos.html)

 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施

**nacos能干什么？**

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。

**nacos使用**

[https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)

nacos实现集群环境下数据的一致性和持久化

多个nacos构成的集群连接在一个mysql数据库上，而不是使用nacos本身自带的数据库，nacos切换数据库mysql

1. 在navicat里面执行nacos-mysql.sql，创建数据库和表
2. 添加配置

```properties
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user=root
db.password=123456
```

nacos集群搭建

1. 在linux下安装mysql

2. 安装jdk

3. 安装nginx

4. 安装nacos

5. 初始化nacos的mysql数据库，创建nacos_config数据库，根据conf目录下面的nacos_config.mysql建表

6. 修改application.properties,添加mysql的连接配置

   ```properties
   spring.datasource.platform=mysql
   db.num=1
   db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
   db.user=root
   db.password=123456
   ```

7. 拷贝cluster.conf文件，添加集群配置 ip:port

8. 拷贝几个nacos结点

9. 修改拷贝后节点的端口号application.properties > server.port=???

   

