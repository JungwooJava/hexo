---
title: 为什么SpringBoot的 jar 可以直接运行？
date: 2021-02-23 09:53:00
tags: [SpringBoot]
---

> ==Spring Boot Loader==提供了一套标准用于执行SpringBoot打包出来的jar



## 打包插件spring-boot-maven-plugin

SpringBoot提供了一个插件spring-boot-maven-plugin用于把程序打包成一个可执行的jar包。在pom文件里加入这个插件即可：

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

打包完生成的executable-jar-1.0-SNAPSHOT.jar内部的结构如下：

```
├── META-INF
│   ├── MANIFEST.MF
│   └── maven
│       └── spring.study
│           └── executable-jar
│               ├── pom.properties
│               └── pom.xml
├── lib
│   ├── aopalliance-1.0.jar
│   ├── classmate-1.1.0.jar
│   ├── spring-boot-1.3.5.RELEASE.jar
│   ├── spring-boot-autoconfigure-1.3.5.RELEASE.jar
│   ├── ...
├── org
│   └── springframework
│       └── boot
│           └── loader
│               ├── ExecutableArchiveLauncher$1.class
│               ├── ...
└── spring
    └── study
        └── executablejar
            └── ExecutableJarApplication.class
```

然后可以直接执行jar包就能启动程序了：

```
java -jar executable-jar-1.0-SNAPSHOT.jar
```

打包出来fat jar内部有4种文件类型：

1. META-INF文件夹：程序入口，其中MANIFEST.MF用于描述jar包的信息
2. lib目录：放置第三方依赖的jar包，比如springboot的一些jar包
3. spring boot loader相关的代码
4. 模块自身的代码

MANIFEST.MF文件的内容：

```
Manifest-Version: 1.0
Implementation-Title: executable-jar
Implementation-Version: 1.0-SNAPSHOT
Archiver-Version: Plexus Archiver
Built-By: Format
Start-Class: spring.study.executablejar.ExecutableJarApplication
Implementation-Vendor-Id: spring.study
Spring-Boot-Version: 1.3.5.RELEASE
Created-By: Apache Maven 3.2.3
Build-Jdk: 1.8.0_20
Implementation-Vendor: Pivotal Software, Inc.
Main-Class: org.springframework.boot.loader.JarLauncher
```

我们看到，它的Main-Class是org.springframework.boot.loader.JarLauncher，当我们使用java -jar执行jar包的时候会调用JarLauncher的main方法，而不是我们编写的SpringApplication。

那么==JarLauncher==这个类是的作用是什么的？

它是SpringBoot内部提供的工具，Spring Boot Loader提供的一个用于执行Application类的工具类(fat jar内部有spring loader相关的代码就是因为这里用到了)。相当于==Spring Boot Loader==提供了一套标准用于执行SpringBoot打包出来的jar



