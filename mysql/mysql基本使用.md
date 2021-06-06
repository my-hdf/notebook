### **创建数据库**

```sql
show databases; #查看数据库
create database db2; 
create database if not exists db3; #创建数据库如果不存在
create database db4 character set utf8; #创建数据库指定字符编码
#创建数据库 如果不存在则创建并设置字符集
create database if not exists db3 character set gbk;
```

