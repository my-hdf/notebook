seata是什么？

[https://seata.io/zh-cn/docs/overview/what-is-seata.html](https://seata.io/zh-cn/docs/overview/what-is-seata.html)

Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。

seata的工作流程

1. TM(事务管理)向TC(事务协调者)申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID;
2. XID在微服务调用链路的上下文中传播
3. RM向TC注册分支事务，将其纳入XID对应全局事务的管辖
4. TM向TC发起针对XID的全局提交或回滚决议
5. TC调度XID下管辖的全部分支事务完成提交或回滚请求

