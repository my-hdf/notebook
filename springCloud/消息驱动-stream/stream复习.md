**消息驱动是什么?**

消息中间件有很多：rabbitmq、kafka、socketmq...

他们的底层有很大差异，学习所有的消息中间件成本很高，也不切实际。Spring Cloud Stream是一个框架，用于构建与共享消息传递系统连接的高度可伸缩的事件驱动微服务。

**消息驱动能干什么？**

屏蔽消息中间件底层的差异，降级切换的成本，使用统一的编程方式。

![image-20210602202638840](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602202638840.png)

-------------------------------------------------------------------------------------------------------

**同一个组的微服务不会重复消费，存在竞争关系；不同组微服务存在重复消费，不存在竞争关系**

**使用分组解决重复消费的问题**

4417和4418处于不同组

![image-20210602212303334](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602212303334.png)

微服务4418(消费服务)

![image-20210602212056328](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602212056328.png)

服务4417(消费服务)

![image-20210602212152458](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602212152458.png)

----------------------------------------------------------------------------------------------

4417和4418处于相同组

配置yaml

4418服务

![image-20210602212546903](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602212546903.png)

4417服务

![image-20210602212628274](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602212628274.png)

rabbitmq

![image-20210602212713891](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602212713891.png)

![image-20210602212756828](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602212756828.png)

发送消息后

4418接收消息

![image-20210602212845849](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602212845849.png)

4417未接收消息

![image-20210602212928602](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602212928602.png)

**当4417和4418处于相同组时，不存在消息重复，因为存在竞争关系**

------------------------------------------------------------------------

使用分组解决消息持久化问题

将消费服务4417和4418关闭，将4417里面的group属性删除。消息的发送方发送4次消息，启动4417，4418

![image-20210602213338721](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602213338721.png)

![image-20210602213352875](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602213352875.png)

输出结果

4417没有接收消息

![image-20210602213454330](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602213454330.png)

4418重新启动正常接收消息，实现消息的持久化

![image-20210602213709187](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210602213709187.png)

**使用分组可以实现消息的持久化**

------------------------------------------------------------------------------

