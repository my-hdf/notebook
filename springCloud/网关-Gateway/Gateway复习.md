gateway和zuul

![image-20210531155359222](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20210531155359222.png)

**gateway核心为路由转发和执行过滤器链**



在构建gateway微服务时，碰到一个错误。这个错误导致的原因是spring-boot的版本影响

```java
Caused by: java.lang.NoClassDefFoundError: reactor/netty/http/server/WebsocketServerSpec$Builder
	at java.lang.Class.getDeclaredMethods0(Native Method) ~[na:1.8.0_131]
	at java.lang.Class.privateGetDeclaredMethods(Class.java:2701) ~[na:1.8.0_131]
	at java.lang.Class.getDeclaredMethods(Class.java:1975) ~[na:1.8.0_131]
	at org.springframework.util.ReflectionUtils.getDeclaredMethods(ReflectionUtils.java:459) ~[spring-core-5.2.0.RELEASE.jar:5.2.0.RELEASE]
	... 21 common frames omitted
Caused by: java.lang.ClassNotFoundException: reactor.netty.http.server.WebsocketServerSpec$Builder
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381) ~[na:1.8.0_131]
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424) ~[na:1.8.0_131]
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:335) ~[na:1.8.0_131]
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357) ~[na:1.8.0_131]
	... 25 common frames omitted
```

