1. **下载mysql8.x.x ，解压到文件夹**
2. **根目录没有data文件夹和my.ini文件，我们也不要创建。**
3. **将“根目录/bin”路径添加到环境变量中，如果不添加就每次执行命令的时候都要带路径，因为我添加了，所以我是不带路径的。**
4. **用管理员启动命令行，win10是右键左下角win图标，选择“Windows powershell（管理员）”。**

5. **输入mysqld --initialize-insecure--user=mysql**
6. **然后输入mysqld --install mysql**
7. **net start mysql启动数据库** 
8. **mysql -uroot -p 这时候要输入密码，因为没有密码默认按回车即可进入mysql，但是不能直接输入mysql进入数据库，一定要mysql -uroot -p回车然后再回车。**
9. **以上操作之后，是不能用最新的Navicat for SQL建立链接的，好像是加密方式不同导致的，一下操作可以设立密码+修复不能用Navicat的问题：**

10. **进入mysql后，下列操作可以建立密码，并且实现Navicat链接： ALTER USER 'root'@'localhost' IDENTIFIED BY '密码' PASSWORD EXPIRE NEVER;**
11. **ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '密码';** 
12. **FLUSH PRIVILEGES;**

