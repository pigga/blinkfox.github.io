最近在使用Maven时，由于对其不太熟的原因，老是出问题。有问题就要总结啦。
问题：
```
 org.apache.catalina.core.StandardContext listenerStart
严重: Error configuring application listener of class org.springframework.web.context.ContextLoaderListener
java.lang.ClassNotFoundException: org.springframework.web.context.ContextLoaderListener
	at org.apache.catalina.loader.WebappClassLoader.loadClass(WebappClassLoader.java:1645)
	at org.apache.catalina.loader.WebappClassLoader.loadClass(WebappClassLoader.java:1491)
	at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4078)
	at org.apache.catalina.core.StandardContext.start(StandardContext.java:4630)
	at org.apache.catalina.core.ContainerBase.start(ContainerBase.java:1045)
	at org.apache.catalina.core.StandardHost.start(StandardHost.java:785)
	at org.apache.catalina.core.ContainerBase.start(ContainerBase.java:1045)
	at org.apache.catalina.core.StandardEngine.start(StandardEngine.java:445)
	at org.apache.catalina.core.StandardService.start(StandardService.java:519)
	at org.apache.catalina.core.StandardServer.start(StandardServer.java:710)
	at org.apache.catalina.startup.Catalina.start(Catalina.java:581)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
	at java.lang.reflect.Method.invoke(Unknown Source)
	at org.apache.catalina.startup.Bootstrap.start(Bootstrap.java:289)
	at org.apache.catalina.startup.Bootstrap.main(Bootstrap.java:414)
    org.apache.catalina.core.StandardContext listenerStart
```
 在一顿搜索之后发现实际上在Web-Inf文件夹下没有lib文件夹。没有该文件夹，项目还怎么运行啊。<br/>
原因：在将MAven项目转化为eclipse下的web项目时，没有把对应的jar转化过去。<br/>
解决办法：选中项目---->properties-->Deployment在该界面把相关的jar添加进去。<br/>
finish.

**主要参考**: http://snowolf.iteye.com/blog/1627343