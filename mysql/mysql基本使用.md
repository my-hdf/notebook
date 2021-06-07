### **数据库基本操作**

1、数据库创建

```sql
show databases; #查看数据库
create database db2; 
create database if not exists db3; #创建数据库如果不存在
create database db4 character set utf8; #创建数据库指定字符编码
#创建数据库 如果不存在则创建并设置字符集
create database if not exists db3 character set gbk;
```

2、数据库字符集修改

```mysql
alter database xxx character set 编码类型
```

3、删除数据库

```mysql
drop database 数据库名称
drop database if exists 数据库名
```

**数据表的基本操作**

```mysql
use mysql #切换数据库
show tables
desc 表名
show table status from 数据库 like '表明' #查看数据库某张表表状态
```

