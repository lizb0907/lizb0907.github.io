---
layout: post
title: MySQL utf8 vs utf8mb4
categories: Problem
description: mysql字符集设置utf8 和 utf8mb4区别
keywords: MySQL utf8 vs utf8mb4
---

mysql字符集设置utf8 和 utf8mb4区别

**目录**

* TOC
{:toc}

## 1.问题引起

```sh
mysql5.7版本，字符集设置为utf8，在执行emoji表情符（4字节的表情字符串）报错误：
   sql execute failure, exception:Incorrect string value: '\xF0\x9F\x98\x86\xF0\x9F...' for 
   column 'content' at row 1, code:1366, sql stat:HY000

原因:
     .MySQL的“utf8”实际上不是真正的UTF-8，只支持每个字符最多三个字节，而真正的UTF-8是每个字符最多四个字节。
     .MySQL的“utf8mb4”才是真正的“UTF-8”
```

## 2.utf8和utf8mb4区别

```sh
参考链接：
  https://stackoverflow.com/questions/30074492/what-is-the-difference-between-utf8mb4-and-utf8-charsets-in-mysql
  
  总结：
     So the character set "utf8"/"utf8mb3" cannot store all Unicode code points: it only supports the range 0x000 to 0xFFFF。
     if you want your column to support storing characters lying outside the BMP (and you usually want to), such as emoji, use "utf8mb4".
```

## 3.从utf8改为utf8mb4可能的影响

```sh
参考链接：
  https://www.eversql.com/mysql-utf8-vs-utf8mb4-whats-the-difference-between-utf8-and-utf8mb4/

  总结（不同版本不一样）：
    1.需要注意有些版本 MySQL Innodb 的索引长度限制为767字节，utf8编码最多占3个字节，对于varchar(255)这样的数据库结构，
      所以255*3=765，没有超出767的限制。
    2.而对于utf8mb4编码，一个字最多可以占用4个字节，那255*4就会超出767的字符限制了。
```

## 4.utf8和utf8mb4性能比较（mysql5.7）

```sh
官网参考链接：
  https://forums.mysql.com/read.php?22,650248,650632

  关于性能对比，mysql5.7版本官网介绍了：  
    1.如果只是将字符存储到表不进行排序，那么utf8 and utf8mb4性能是没有区别的。（咱们游戏业务基本不涉及数据库排序）
    2.如果涉及到排序，utf8 默认使用utf8_general_ci ,utf8mb4默认使用utf8mb4_general_ci， 这两种排序方式都使用一般排序方式，没啥区别。

结论：mysql5.7utf8 和 utf8mb4性能在游戏业务上基本没啥区别。
```

## 5.设置utf8mb4存储emoji表情遇到的问题

```sh
1.一般需要设置数据库编码是utf8mb4，同时数据库链接url中设置characterEncoding=utf8。（不同版本可能不同）

2.pom.xml中的mysql-connector-java版本号需要符合。
```

pom.xml中的mysql-connector-java版本号需要符合如下：
![](/images/posts/problem/1.jpg)


## 6.未来趋势

```sh
mysql团队发布的：
   http://mysqlserverteam.com/mysql-8-0-when-to-use-utf8mb3-over-utf8mb4/

   总结：
      1.mysql8.0以后的版本， 性能utf8mb4大于utf8。
      2.mysql 8.0默认使用 utf8mb4， utf-8标识为废弃。
      3.未来版本utf-8将会被废弃。
```