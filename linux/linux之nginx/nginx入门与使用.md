### nginx基础使用

#### **安装nginx前准备**

1. 虚拟机centos7.x

2. 永久关闭防火墙  systemctl disable firewalld

3. 关闭selinux安全系统 目的：使得之后的配置更加简单

   ```shell
   vim /etc/selinux/config
   #设置SELINUX=disabled
   reboot #重启
   sestatus #查看状态
   ```

#### **下载安装nginx**

方式1

```sheLl
#下载
wget http://nginx.org/download/nginx-1.20.1.tar.gz
# 安装编译工具
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
#解压
tar -zxvf nginx-1.8.1.tar.gz -C /opt/module/
cd /opt/module/nginx-1.8.1
#执行configure文件
./configure
#编译安装
make
make install
whereis nginx
cd /usr/local/nginx
cd sbin/
./nginx
# 通过ip访问 默认80端口无需配置

```

方式2

[http://nginx.org/en/linux_packages.html#RHEL-CentOS](http://nginx.org/en/linux_packages.html#RHEL-CentOS)

启用和关闭nginx

1、信号关闭

kill -signal pid

|     信号     |                    作用                    |
| :----------: | :----------------------------------------: |
| **TERM/INT** |            **立即关闭整个服务**            |
|   **QUIT**   |           **优雅的关闭整个服务**           |
|   **HUP**    |              **刷新配置信息**              |
|   **USR1**   | **重新打开日志文件，可以用来进行日志分割** |
|   **USR2**   |           **切换到最新的nginx**            |
|  **winch**   |   **停止接收所有请求，关闭所有work进程**   |

2、使用命令

进入到nginx安装目录，进入sbin

nginx -h 查看nginx相关命令

```shell
Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/nginx/)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file
```

```shell
nginx -s stop #直接关闭
nginx -s quit #优雅关闭
nginx -s reload #修改配置文件后 重新启动
nginx -c 配置文件地址
```

#### nginx升级

1、make方式升级

```shell
#下载好最新的nginx 解压 编译
./configure
make #不需要make install 安装
# 到nginx 安装目录 /usr/local/nginx/sbin
mv nginx nginxold #重命名
# 到新版nginx目录下 拷贝nginx二进制可执行文件 在objs下
cp nginx /usr/local/nginx/sbin
# 回到新版nginx目录
make upgrade
```

2、使用信号升级

```shell
#下载好最新的nginx 解压 编译
./configure
make #不需要make install 安装
# 到nginx的安装目录 /usr/local/nginx/sbin
mv nginx nginxold
# 将新版本nginx的二进制启动文件拷贝进入 /usr/local/nginx/sbin下
cp nginx /usr/local/nginx/sbin
# 查看进程
ps -ef | grep nginx
# 发送信号 平滑升级
kill -USR2 master进程号
# 停止之前的master进程
kill -QUIT `cat /usr/local/nginx/logs/nginx.pid.oldbin`
```

#### **配置文件nginx.conf**

**nginx全局块user指令**

使用user指令可以指定启动工作进程的用户及用户组，这样对于系统的访问更加精细、更加安全。

```shell
#修改配置nginx.conf文件
user www  #指定某用户
# 检查配置文件
 ./nginx -t
nginx: [emerg] getpwnam("www") failed in /usr/local/nginx/conf/nginx.conf:2
nginx: configuration file /usr/local/nginx/conf/nginx.conf test failed # 不存在www用户
useradd www
# 检查配置文件
./nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
#修改配置文件
 location / {
           #指定需要加载文件的路径
            root  /home/www/html;
            index  index.html index.htm;
           # proxy_pass http://cluster;
        }
./nginx -s reload
```

http://39.103.228.36/