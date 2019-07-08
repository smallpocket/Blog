---
title: Jooq：Start
type: tags
tags:
  - null
date: 2019-07-04 11:12:50
categories:
description:
---

# Jooq

## 示例

### 创建数据库

```sql
DATABASE `library`;
 
USE `library`;
 
CREATE TABLE `author` (
  `id` int NOT NULL,
  `first_name` varchar(255) DEFAULT NULL,
  `last_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
);
 
INSERT INTO `author` (`id`, `first_name`, `last_name`) VALUES ('1', '3', 'zhang'), ('2', '4', 'li');
```

***sql语句***

```sql
SELECT AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME, COUNT(*)
    FROM AUTHOR
    JOIN BOOK ON AUTHOR.ID = BOOK.AUTHOR_ID
   WHERE BOOK.LANGUAGE = 'DE'
     AND BOOK.PUBLISHED > DATE '2008-01-01'
GROUP BY AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME
  HAVING COUNT(*) > 5
ORDER BY AUTHOR.LAST_NAME ASC NULLS FIRST
   LIMIT 2
   OFFSET 1
```

***Java***

```java
create.select(AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME, count())
      .from(AUTHOR)
      .join(BOOK).on(AUTHOR.ID.equal(BOOK.AUTHOR_ID))
      .where(BOOK.LANGUAGE.eq("DE"))
      .and(BOOK.PUBLISHED.gt(date("2008-01-01")))
      .groupBy(AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
      .having(count().gt(5))
      .orderBy(AUTHOR.LAST_NAME.asc().nullsFirst())
      .limit(2)
      .offset(1)
```

### 映射

这里，要使用jOOQ的命令行工具生成映射到author表的Java类。

代码生成的最简单的方法是将jOOQ的3个jar文件和MySQL Connector jar文件复制到一个临时目录（本示例中目录是test-generated）， 然后创建一个如下所示的library.xml（名字随意修改）：

目录结构：

![1562227671175](assets/1562227671175.png)

library.xml配置文件内容：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration xmlns="http://www.jooq.org/xsd/jooq-codegen-3.9.2.xsd">
    <!-- Configure the database connection here -->
    <jdbc>
        <driver>com.mysql.jdbc.Driver</driver>
        <!-- 数据库url -->
        <url>jdbc:mysql://localhost:3306/library?useUnicode=true&amp;characterEncoding=UTF-8</url>
        <!-- 数据库账号 -->
        <user>root</user>
        <!-- 数据库账号密码 -->
        <password>root</password>
    </jdbc>
 
    <generator>
        <!-- The default code generator. You can override this one, to generate your own code style.
             Supported generators:
             - org.jooq.util.JavaGenerator
             - org.jooq.util.ScalaGenerator
             Defaults to org.jooq.util.JavaGenerator -->
        <name>org.jooq.util.JavaGenerator</name>
 
        <database>
            <!-- The database type. The format here is:
                 org.util.[database].[database]Database -->
            <name>org.jooq.util.mysql.MySQLDatabase</name>
 
            <!-- The database schema (or in the absence of schema support, in your RDBMS this
                 can be the owner, user, database name) to be generated -->
            <inputSchema>library</inputSchema>
 
            <!-- All elements that are generated from your schema
                 (A Java regular expression. Use the pipe to separate several expressions)
                 Watch out for case-sensitivity. Depending on your database, this might be important! -->
            <includes>.*</includes>
 
            <!-- All elements that are excluded from your schema
                 (A Java regular expression. Use the pipe to separate several expressions).
                 Excludes match before includes, i.e. excludes have a higher priority -->
            <excludes></excludes>
        </database>
 
        <target>
            <!-- The destination package of your generated classes (within the destination directory) -->
            <!-- 生成的包名，生成的类在此包下 -->
            <packageName>test.generated</packageName>
 
            <!-- The destination directory of your generated classes. Using Maven directory layout here -->
            <!-- 输出的目录 -->
            <directory>J:\IDEA_Work_Space\maven_jooq_demo\src\main\java</directory>
        </target>
    </generator>
</configuration>

```

cd到目录下

```shell
java -classpath jooq-3.9.5.jar;jooq-meta-3.9.5.jar;jooq-codegen-3.9.5.jar;mysql-connector-java-5.1.41.jar; org.jooq.util.GenerationTool library.xml
```

### 连接数据库

```Java
package com.jooq.cn;
 
import org.jooq.DSLContext;
import org.jooq.Record;
import org.jooq.Result;
import org.jooq.SQLDialect;
import org.jooq.impl.DSL;
 
import java.sql.Connection;
import java.sql.DriverManager;
 
import static test.generated.tables.Author.AUTHOR;
 
public class Main {
 
    public static void main(String[] args) {
 
        //用户名
        String userName = "root";
        //密码
        String password = "root";
        //mysql链接url
        String url = "jdbc:mysql://localhost:3306/library";
        Connection conn;
        try {
             //这是JDBC Mysql连接
             conn = DriverManager.getConnection(url, userName, password); 
             //基于JOOQ实现的简单查询
             //传入Connection连接对象、数据方言得到一个DSLContext的实例，然后使用DSL对象查询得到一个Result对象。
            DSLContext using = DSL.using(conn, SQLDialect.MYSQL);
            Result<Record> fetch = using.select().from(AUTHOR).fetch();
 
 
            //for循环输出结果
            for (Record record : fetch) {
                Integer id = record.getValue(AUTHOR.ID);
                String firstName = record.getValue(AUTHOR.FIRST_NAME);
                String lastName = record.getValue(AUTHOR.LAST_NAME);
                /**
                 * 控制台输出
                 * ID: 1 first name: 3 last name: zhang
                 * ID: 2 first name: 4 last name: li
                 */
                System.out.println("ID:"+id + "firstName"+ firstName + "lastName: " + lastName);
            }
            //关闭连接
            conn.close();
 
        }catch (Exception e){
            e.printStackTrace();
        }
 
    }
}

```



### 关闭数据库连接

DSLContext不会主动关闭连接，需要我们手动关闭。

# 参考 #

1. 

