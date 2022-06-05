---
layout: post
title: Instrumentation项目接入使用
categories: Mmo-Game
description: Instrumentation项目接入使用
keywords: Instrumentation，项目，使用，接入
---

Instrumentation项目接入使用

**目录**

* TOC
{:toc}

## 一：新建Agent工程

### 1.com.minigame.reload 目录下新建 ClassReloader 类

```java
package com.minigame.reload;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

import java.lang.instrument.ClassDefinition;
import java.lang.instrument.Instrumentation;

public class ClassReloader
{
    private static final Logger BP_RELOAD = LogManager.getLogger("reload");
    private static Instrumentation inst = null;

    public ClassReloader()
    {
    }

    public static void premain(String agentArgs, Instrumentation i)
    {
        inst = i;
        if (inst != null)
        {
            BP_RELOAD.info("Instrumentation 初始化成功");
        } else
        {
            BP_RELOAD.error("Instrumentation 初始化失败");
        }

    }

    public static int redefineClass(ClassDefinition classDefinition)
    {
        if (classDefinition != null && inst != null)
        {
            try
            {
                inst.redefineClasses(new ClassDefinition[]{classDefinition});
            } catch (Exception var2)
            {
                BP_RELOAD.error("热更失败:{}", ExceptionUtils.exceptionToString(var2));
                return -1;
            }

            BP_RELOAD.info("热更成功");
            return 0;
        } else
        {
            return -1;
        }
    }
}
```

### 2.com.minigame.reload 目录下新建 ExceptionUtils 类

```java
package com.minigame.reload;

public class ExceptionUtils
{
    private static final int stackLength = 10;

    public ExceptionUtils()
    {
    }

    public static String exceptionToString(Exception e)
    {
        StackTraceElement[] sts = e.getStackTrace();
        StringBuilder sb = new StringBuilder();
        int len = sts.length > 10 ? 10 : sts.length;

        for (int i = 0; i < len; ++i)
        {
            sb.append(sts[i]);
            sb.append("\n");
        }

        return sb.toString();
    }
}
```

### 3.利用mvn自动打包（省去人工配置META-INF文件）, POM.xml配置如下

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>agent</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>agent</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-api</artifactId>
      <version>2.8.2</version>
    </dependency>

    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>2.8.2</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>2.3.1</version>
        <configuration>
          <archive>
            <manifest>
              <addClasspath>true</addClasspath>
            </manifest>
            <manifestEntries>
              <Premain-Class>
                com.minigame.reload.ClassReloader
              </Premain-Class>
              <Can-Redefine-Classes>true</Can-Redefine-Classes>
              <Can-Retransform-Classes>true</Can-Retransform-Classes>
            </manifestEntries>
          </archive>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```
打包成jar,  将jar重命名为agent-1.jar



## 二：项目引入agent-1.jar使用

### 1.将agent-1.jar放到resources的class文件夹下

```sh
1.在resources目录下，新建class文件夹。
2.class文件夹引入agent-1.jar
```

### 2.项目启动参数配置VM参数调用agent-1.jar

```sh
-javaagent:H:\project\minigame\minigame\minigame.server\src\main\resources\class\agent-1.jar

配置上自己项目的jar路径
```

### 3.项目pom.xml配置引入本地agent-1.jar

```java
<dependency>
			<groupId>com.minigame</groupId>
			<artifactId>agent</artifactId>
			<version>1.0</version>
			<scope>system</scope>
			<systemPath>${pom.basedir}/src/main/resources/class/agent-1.jar</systemPath>
</dependency>


这一步非常重要，配置后项目可以引入ClassReloader使用。
```


### 4.在需要用的类下直接导入ClassReloader类使用

```sh
在类下导入类：
import com.minigame.reload.ClassReloader;

然后，直接就可以使用：
int result = ClassReloader.redefineClass(classDefinition);
if (result < 0)
{
    throw new Exception("重定义生成的dispatcher代码失败");
}

非常方便！
```










