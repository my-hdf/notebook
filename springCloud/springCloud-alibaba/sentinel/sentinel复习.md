**手册中文官网**

[https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D](https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D)

**sentinel是什么？**

sentinel是以流量为切入点，从流量控制、流量整形、熔断控制、系统负载等对各维度保持系统的稳定性。

sentinel怎么用？

1. 下载sentinel-dashboard.jar文件，并启动，浏览器访问localhost:8080可以看到一个流量监控的一个可视化界面

2. 在需要进行监控的微服务里面添加一个依赖

   ```xml
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
   </dependency>
   ```

3. 添加配置信息

   ```yml
   spring:
     cloud:
       nacos:
         server-addr: localhost:8848
       sentinel:
         transport:
           dashboard: localhost:8080
           port: 8719
   ```

sentinel客户端的基本使用

[https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6#%E5%9F%BA%E4%BA%8Eqps%E5%B9%B6%E5%8F%91%E6%95%B0%E7%9A%84%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6](https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6#%E5%9F%BA%E4%BA%8Eqps%E5%B9%B6%E5%8F%91%E6%95%B0%E7%9A%84%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6)





**流控规则**





QPS:每秒钟处理的请求次数

线程数：处理请求的线程个数，

-------------------------------------------------------

**warm up预热**：指的是处理请求的数量经过一段时间从某一个值(阈值/3)到达最大值(阈值)，无法处理的请求将会进行降级处理

**直接快速失败：**

1. QPS快速失败

   当请求个数超过设定的阈值，进行降级处理

2. 线程快速失败

   当请求太多，处理请求的线程无法处理请求时，进行降级处理

**关联**：A请求关联B请求，处理B请求的资源到达了最大限度，A请求会进行限流

**排队**：排队处理每一个请求，每个请求都会被处理，最大限度的利用了空闲资源







-------------------------------------------------------------

**降级规则**

[https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7](https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7)

- **慢调用比例 (`SLOW_REQUEST_RATIO`)**：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。
- **异常比例 (`ERROR_RATIO`)**：当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。
- **异常数 (`ERROR_COUNT`)**：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。

