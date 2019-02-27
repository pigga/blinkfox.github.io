今天在项目开发过程中遇到的比较诡异的问题之
```
[org.hibernate.util.JDBCExceptionReporter] - Cannot load JDBC driver class 'net.sourceforge.jtds.jdbc.Driver'。
```
**解决办法**，在项目部署的tomcat的lib文件夹中加入jtds.jar解决问题。但是后来在同事那边发现，他那边tomcat的lib文件夹中并无该jar但仍正常运行。难道这是传说中的人品积累？求知道的人说一些具体原因。