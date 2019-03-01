---
title: maven-assembly-plugin报错
date: 2018-08-30 13:57:00 +0800
layout: post
permalink: /blog/2018/08/30/maven-assembly-plugin报错.html
categories:
  - JAVA
tags:
  - JAVA
  - Maven
---
Maven pom.xml保存
```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-assembly-plugin:2.5.5:single (default) on project ycsb: Failed to create assembly: Artifact: com.yahoo.ycsb:riak-binding:jar:0.3.1-RC1-SNAPSHOT (included by module) does not have an artifact with a file. Please ensure the package phase is run before the assembly is generated. -> [Help 1]
org.apache.maven.lifecycle.LifecycleExecutionException: Failed to execute goal org.apache.maven.plugins:maven-assembly-plugin:2.5.5:single (default) on project ycsb: Failed to create assembly: Artifact: com.yahoo.ycsb:riak-binding:jar:0.3.1-RC1-SNAPSHOT (included by module) does not have an artifact with a file. Please ensure the package phase is run before the assembly is generated.
```
在新建的Maven项目里遇到如上问题，使用stackoverflow上大神的解决方案一下就ok。<br/>
替换或新增如下内容
```
<plugin>
  <artifactId>maven-assembly-plugin</artifactId>
  <configuration>
    <archive>
      <manifest>
        <mainClass>com.test.MainClassName</mainClass>
      </manifest>
    </archive>
    <descriptorRefs>
      <descriptorRef>jar-with-dependencies</descriptorRef>
    </descriptorRefs>
  </configuration>
  <executions>
    <execution>
      <id>make-assembly</id> 
      <phase>package</phase> <!-- packaging phase -->
      <goals>
        <goal>single</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```