---
title: Jooq：文档
type: tags
tags:
  - null
date: 2019-07-07 18:04:32
categories:
description:
---

# Jooq

## 提出问题

## 为什么要用（作用）

### 与JPA相比

到目前为止，只有少数的数据库抽象框架或者库，真正尊重SQL作为语言的一等公民，而包括JPA、EJB、doubts等许多框架都试图隐藏SQL本身，将其范围最小化。

JOOQ填补了这个空白

### 与LINQ相比

### 与SQL相比

SQL可以作为纯文本编写并通过JDBC API传递，多年来，人们对这种方式保持警惕

- 没有类型安全
- 没有语法安全
- 没有绑定值索引安全性
- 详细的SQL字符串连接
- 无聊的绑定值索引技术
- JDBC中的详细资源和异常处理
- 一个非常有“状态”的，不是非常面向对象的jdbc api，很难使用

# JOOQ前置了解

## 代码块内容

若未给出假设，则假定使用的是Oracle语法

当看到独立函数，则是从org.jooq.impl.DSL导入的静态函数。

```java
每当您看到BOOK / Book，AUTHOR / Author和类似实体时，假设它们是从生成的模式中导入的（静态的）
BOOK.TITLE，AUTHOR.LAST_NAME //对应com.example.generated.Tables.BOOK.TITLE，com.example.generated.Tables.BOOK.TITLE
FK_BOOK_AUTHOR //对应com.example.generated.Keys.FK_BOOK_AUTHOR
```

```java
//每当看到Java代码中使用“create”时，假设这是org.jooq.DSLContext的一个实例。
//它被称为“创建”的原因是，从DSL对象创建了一个jOOQ QueryPart。
//“create”因此是非静态查询DSL的入口点
DSLContext create = DSL.using（connection，SQLDialect.ORACLE）;
```

## 执行

JOOQ无法确定声明是否完整，直到执行fetch或者execute

```java
create.selectOne().fetch();
create.update(T).set(T.V, 1).execute();
```

## [设置](https://www.jooq.org/doc/3.10/manual/getting-started/the-manual/#N3A272)

jOOQ允许使用[org.jooq.conf.Settings](https://www.jooq.org/javadoc/3.10.x/org/jooq/conf/Settings.html)覆盖运行时行为。如果未指定任何内容，则假定使用默认运行时设置。

## [样本数据库](https://www.jooq.org/doc/3.10/manual/getting-started/the-manual/#N3A27B)

jOOQ查询示例针对示例数据库运行。

### 示例数据库

```sql
CREATE TABLE language (
  id              NUMBER(7)     NOT NULL PRIMARY KEY,
  cd              CHAR(2)       NOT NULL,
  description     VARCHAR2(50)
);

CREATE TABLE author (
  id              NUMBER(7)     NOT NULL PRIMARY KEY,
  first_name      VARCHAR2(50),
  last_name       VARCHAR2(50)  NOT NULL,
  date_of_birth   DATE,
  year_of_birth   NUMBER(7),
  distinguished   NUMBER(1)
);

CREATE TABLE book (
  id              NUMBER(7)     NOT NULL PRIMARY KEY,
  author_id       NUMBER(7)     NOT NULL,
  title           VARCHAR2(400) NOT NULL,
  published_in    NUMBER(7)     NOT NULL,
  language_id     NUMBER(7)     NOT NULL,
  
  CONSTRAINT fk_book_author     FOREIGN KEY (author_id)   REFERENCES author(id),
  CONSTRAINT fk_book_language   FOREIGN KEY (language_id) REFERENCES language(id)
);

CREATE TABLE book_store (
  name            VARCHAR2(400) NOT NULL UNIQUE
);

CREATE TABLE book_to_book_store (
  name            VARCHAR2(400) NOT NULL,
  book_id         INTEGER       NOT NULL,
  stock           INTEGER,
  
  PRIMARY KEY(name, book_id),
  CONSTRAINT fk_b2bs_book_store FOREIGN KEY (name)        REFERENCES book_store (name) ON DELETE CASCADE,
  CONSTRAINT fk_b2bs_book       FOREIGN KEY (book_id)     REFERENCES book (id)         ON DELETE CASCADE
);
```

### 示例数据

```sql
INSERT INTO language (id, cd, description) VALUES (1, 'en', 'English');
INSERT INTO language (id, cd, description) VALUES (2, 'de', 'Deutsch');
INSERT INTO language (id, cd, description) VALUES (3, 'fr', 'Français');
INSERT INTO language (id, cd, description) VALUES (4, 'pt', 'Português');

INSERT INTO author (id, first_name, last_name, date_of_birth    , year_of_birth)
  VALUES           (1 , 'George'  , 'Orwell' , DATE '1903-06-26', 1903         );
INSERT INTO author (id, first_name, last_name, date_of_birth    , year_of_birth)
  VALUES           (2 , 'Paulo'   , 'Coelho' , DATE '1947-08-24', 1947         );

INSERT INTO book (id, author_id, title         , published_in, language_id)
  VALUES         (1 , 1        , '1984'        , 1948        , 1          );
INSERT INTO book (id, author_id, title         , published_in, language_id)
  VALUES         (2 , 1        , 'Animal Farm' , 1945        , 1          );
INSERT INTO book (id, author_id, title         , published_in, language_id)
  VALUES         (3 , 2        , 'O Alquimista', 1988        , 4          );
INSERT INTO book (id, author_id, title         , published_in, language_id)
  VALUES         (4 , 2        , 'Brida'       , 1990        , 2          );

INSERT INTO book_store VALUES ('Orell Füssli');
INSERT INTO book_store VALUES ('Ex Libris');
INSERT INTO book_store VALUES ('Buchhandlung im Volkshaus');

INSERT INTO book_to_book_store VALUES ('Orell Füssli'             , 1, 10);
INSERT INTO book_to_book_store VALUES ('Orell Füssli'             , 2, 10);
INSERT INTO book_to_book_store VALUES ('Orell Füssli'             , 3, 10);
INSERT INTO book_to_book_store VALUES ('Ex Libris'                , 1, 1 );
INSERT INTO book_to_book_store VALUES ('Ex Libris'                , 3, 2 );
INSERT INTO book_to_book_store VALUES ('Buchhandlung im Volkshaus', 3, 1 );
```

## 不同的使用情况进行JOOQ

### JOOQ作为SQL构建器

允许为任何数据库构建有效的SQL，使用JOOQ的查询DSL API将字符串、文字和其他用户定义的对象包装到面向对象、类型安全的AST当中，为SQL建模

```java
// Fetch a SQL string from a jOOQ Query in order to manually execute it with another tool.
// For simplicity reasons, we're using the API to construct case-insensitive object references, here.
String sql = create.select(field("BOOK.TITLE"), field("AUTHOR.FIRST_NAME"), field("AUTHOR.LAST_NAME"))
                   .from(table("BOOK"))
                   .join(table("AUTHOR"))
                   .on(field("BOOK.AUTHOR_ID").eq(field("AUTHOR.ID")))
                   .where(field("BOOK.PUBLISHED_IN").eq(1948))
                   .getSQL();
```

之后，该sql语句可以直接使用JDBC进行执行，

### JOOQ作为具有代码生成的SQL构建器

### JOOQ作为SQL执行器

使用JOOQ直接执行JOOQ生成的SQL语句，通过让JOOQ执行SQL，JOOQ查询DSL成为真正的嵌入式SQL

```java
// Typesafely execute the SQL statement directly with jOOQ
Result<Record3<String, String, String>> result =
create.select(BOOK.TITLE, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
      .from(BOOK)
      .join(AUTHOR)
      .on(BOOK.AUTHOR_ID.eq(AUTHOR.ID))
      .where(BOOK.PUBLISHED_IN.eq(1948))
      .fetch();
```

并且可以使用任何的SQL构建工具构建SQL语句，之后由JOOQ运行SQL语句。

```java
// Use your favourite tool to construct SQL strings:
String sql = "SELECT title, first_name, last_name FROM book JOIN author ON book.author_id = author.id " +
             "WHERE book.published_in = 1984";

// Fetch results using jOOQ
Result<Record> result = create.fetch(sql);

// Or execute that SQL with JDBC, fetching the ResultSet with jOOQ:
ResultSet rs = connection.createStatement().executeQuery(sql);
Result<Record> result = create.fetch(rs);
```

### JOOQ for CURD

```java
// Fetch an author
AuthorRecord author : create.fetchOne(AUTHOR, AUTHOR.ID.eq(1));

// Create a new author, if it doesn't exist yet
if (author == null) {
    author = create.newRecord(AUTHOR);
    author.setId(1);
    author.setFirstName("Dan");
    author.setLastName("Brown");
}

// Mark the author as a "distinguished" author and store it
author.setDistinguished(1);

// Executes an update on existing authors, or insert on new ones
author.store();
```

### PRO的JOOQ

## JOOQ的工具

- [jOOQ的执行监听器](https://www.jooq.org/doc/3.10/manual/sql-execution/execute-listeners/)：jOOQ允许您将自定义执行监听器挂钩到jOOQ的SQL语句执行生命周期中，以集中协调对正在执行的SQL执行的任何操作。用于记录，标识生成，SQL跟踪，性能测量等。
- [记录](https://www.jooq.org/doc/3.10/manual/sql-execution/logging/)：jOOQ内置一个标准的DEBUG记录器，用于记录和跟踪所有已执行的SQL语句和获取的结果集
- [存储过程](https://www.jooq.org/doc/3.10/manual/sql-execution/stored-procedures/)：jOOQ支持您喜欢的数据库的存储过程和函数。生成所有例程和用户定义类型，并且可以作为函数引用包含在jOOQ的SQL构建API中。
- [批量执行](https://www.jooq.org/doc/3.10/manual/sql-execution/batch-execution/)：执行大量SQL语句时，批量执行很重要。与JDBC相比，jOOQ简化了这些操作
- [导出](https://www.jooq.org/doc/3.10/manual/sql-execution/exporting/)和[导入](https://www.jooq.org/doc/3.10/manual/sql-execution/importing/)：jOOQ附带API，可以轻松导出/导入各种格式的数据

# JOOQ教程入门

## JOOQ简单7个步骤

### 准备

JOOQ的***maven***

```xml
<dependency>
  <groupId>org.jooq</groupId>
  <artifactId>jooq</artifactId>
  <version>3.10.8</version>
</dependency>
<dependency>
  <groupId>org.jooq</groupId>
  <artifactId>jooq-meta</artifactId>
  <version>3.10.8</version>
</dependency>
<dependency>
  <groupId>org.jooq</groupId>
  <artifactId>jooq-codegen</artifactId>
  <version>3.10.8</version>
</dependency>
```

[商业版](<https://www.jooq.org/doc/3.10/manual/getting-started/tutorials/jooq-in-7-steps/jooq-in-7-steps-step1/>)

### 数据库

创建一个library的数据库以及一个对应的author表

```sql
CREATE DATABASE `library`;

USE `library`;

CREATE TABLE `author` (
  `id` int NOT NULL,
  `first_name` varchar(255) DEFAULT NULL,
  `last_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

### 代码生成

使用JOOQ的命令行工具生成映射到我们刚创建的Author表的类。详细信息参照：[关于设置代码生成器的jOOQ手册页](https://www.jooq.org/doc/3.10/manual/code-generation/)

最简单的方式是将JOOQ的jar文件（3个）以及数据库的connect文件复制到临时目录下，创建一个library.xml

- 更新用户名、url、driver等数据
- `database.includes`里包含要创建的数据库表，`database.excludes`包含需要排除的数据库表
- `generator.target.package` - 将其设置为要为生成的类创建的父包。
- 设置`test.generated`将导致`test.generated.Author`和`test.generated.AuthorRecord`创建
- `generator.target.directory` - 要输出的目录。

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration xmlns="http://www.jooq.org/xsd/jooq-codegen-3.10.0.xsd">
  <!-- Configure the database connection here -->
  <jdbc>
    <driver>com.mysql.jdbc.Driver</driver>
    <url>jdbc:mysql://localhost:3306/library</url>
    <user>root</user>
    <password></password>
  </jdbc>

  <generator>
    <!-- The default code generator. You can override this one, to generate your own code style.
         Supported generators:
         - org.jooq.util.JavaGenerator
         - org.jooq.util.ScalaGenerator
         Defaults to org.jooq.util.JavaGenerator 
         
         Note the classes have been moved to org.jooq.codegen or org.jooq.meta in jOOQ 3.11 -->
    <name>org.jooq.util.JavaGenerator</name>

    <database>
      <!-- The database type. The format here is:
           org.jooq.util.[database].[database]Database
       
           Note the classes have been moved to org.jooq.codegen or org.jooq.meta in jOOQ 3.11 -->
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
      <packageName>test.generated</packageName>

      <!-- The destination directory of your generated classes. Using Maven directory layout here -->
      <directory>C:/workspace/MySQLTest/src/main/java</directory>
    </target>
  </generator>
</configuration>
```

到达临时目录后，windows键入。记得替换文件名称

```shell
java -classpath jooq-3.10.8.jar; jooq-meta-3.10.8.jar; jooq-codegen-3.10.8.jar; mysql-connector-java-5.1.18-bin.jar;。
  org.jooq.util.GenerationTool library.xml
```

mac键入

```shell
java -classpath jooq-3.10.8.jar：jooq-meta-3.10.8.jar：jooq-codegen-3.10.8.jar：mysql-connector-java-5.1.18-bin.jar：。
  org.jooq.util.GenerationTool library.xml
```

成功后的输出

```java
Nov 1, 2011 7:25:06 PM org.jooq.impl.JooqLogger info
INFO: Initialising properties  : /library.xml
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Database parameters
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: ----------------------------------------------------------
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO:   dialect                : MYSQL
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO:   schema                 : library
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO:   target dir             : C:/workspace/MySQLTest/src
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO:   target package         : test.generated
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: ----------------------------------------------------------
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Emptying                 : C:/workspace/MySQLTest/src/test/generated
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Generating classes in    : C:/workspace/MySQLTest/src/test/generated
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Generating schema        : Library.java
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Schema generated         : Total: 122.18ms
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Sequences fetched        : 0 (0 included, 0 excluded)
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Tables fetched           : 5 (5 included, 0 excluded)
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Generating tables        : C:/workspace/MySQLTest/src/test/generated/tables
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: ARRAYs fetched           : 0 (0 included, 0 excluded)
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Enums fetched            : 0 (0 included, 0 excluded)
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: UDTs fetched             : 0 (0 included, 0 excluded)
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Generating table         : Author.java
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Tables generated         : Total: 680.464ms, +558.284ms
Nov 1, 2011 7:25:07 PM org.jooq.impl.JooqLogger info
INFO: Generating Keys          : C:/workspace/MySQLTest/src/test/generated/tables
Nov 1, 2011 7:25:08 PM org.jooq.impl.JooqLogger info
INFO: Keys generated           : Total: 718.621ms, +38.157ms
Nov 1, 2011 7:25:08 PM org.jooq.impl.JooqLogger info
INFO: Generating records       : C:/workspace/MySQLTest/src/test/generated/tables/records
Nov 1, 2011 7:25:08 PM org.jooq.impl.JooqLogger info
INFO: Generating record        : AuthorRecord.java
Nov 1, 2011 7:25:08 PM org.jooq.impl.JooqLogger info
INFO: Table records generated  : Total: 782.545ms, +63.924ms
Nov 1, 2011 7:25:08 PM org.jooq.impl.JooqLogger info
INFO: Routines fetched         : 0 (0 included, 0 excluded)
Nov 1, 2011 7:25:08 PM org.jooq.impl.JooqLogger info
INFO: Packages fetched         : 0 (0 included, 0 excluded)
Nov 1, 2011 7:25:08 PM org.jooq.impl.JooqLogger info
INFO: GENERATION FINISHED!     : Total: 791.688ms, +9.143ms
```

### 连接到数据库

连接数据库

```java
// For convenience, always static import your generated tables and jOOQ functions to decrease verbosity:
import static test.generated.Tables.*;
import static org.jooq.impl.DSL.*;

import java.sql.*;

public class Main {
    public static void main(String[] args) {
        String userName = "root";
        String password = "";
        String url = "jdbc:mysql://localhost:3306/library";

        // Connection is the only JDBC resource that we need
        // PreparedStatement and ResultSet are handled by jOOQ, internally
        try (Connection conn = DriverManager.getConnection(url, userName, password)) {
            // ...
        } 

        // For the sake of this tutorial, let's keep exception handling simple
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 查询

DSL不会自己关闭，需要我们自己去完成

```Java
DSLContext create = DSL.using(conn, SQLDialect.MYSQL);
Result<Record> result = create.select().from(AUTHOR).fetch();
```

### 迭代

打印结果

```java
for (Record r : result) {
    Integer id = r.getValue(AUTHOR.ID);
    String firstName = r.getValue(AUTHOR.FIRST_NAME);
    String lastName = r.getValue(AUTHOR.LAST_NAME);

    System.out.println("ID: " + id + " first name: " + firstName + " last name: " + lastName);
}
```

### 探索

# 数据库版本管理工具Flyway

## Flyway简介

Flyway是独立于数据库的应用、管理、跟踪数据库变更的数据库版本管理工具。
Flyway的项目主页是：<https://flywaydb.org/>

### 为什么使用Flyway

- 不同的开发人员在开发产品特性时，都有可能更新数据库（添加新表，新的约束等）。当开发人员完成工作并提交代码时，代码会被合并到主分支并在测试服务器上执行单元测试与集成测试。我们在哪个环节来执行数据库的更新操作呢？由QA 部门手工执行sql 脚本？或者我们开发一断程序自动执行数据库更新？以什么顺序来执行这些更新脚本？这些问题同样存在于生产环境。
- 我们的产品部署在不同的客户服务器上，以及很多的测试、联调、实验局、销售环境上。不同的客户和测试环境上都部署着不同版本的产品。当他们需要升级他们的产品到新的版本时，我们不仅需要让他们的管理员可以升级产品到新的版本，同时需要保留他们的已有数据。在升级产品的步骤中，我们清楚地知道客户数据库的当前版本，以及需要在该数据库上执行哪些数据库更新脚本，来更新数据库表结构与数据库中已存在的数据。当升级完成时，数据库表结构及数据应当与升级后的产品版本保持一致。
- 当升级失败时（比如在升级过程中出现网络连接失败），我们应当支持对失败进行修复。

### 更多Flyway文章（参考以下文档）

- [数据库版本管理工具Flyway——基础篇](http://casheen.iteye.com/blog/1749916)
- [Flyway学习笔记](http://blog.csdn.net/tanghin/article/details/51264795)
- [官方文档](https://flywaydb.org/documentation/)

## Maven配置

在pom文件当中定义如下属性，以便在插件配置中进行重用

```xml
<properties>
    <db.url>jdbc:h2:~/flyway-test</db.url>
    <db.username>sa</db.username>
</properties>
```

### maven项目配置

***依赖***

```xml
<!-- We'll add the latest version of jOOQ and our JDBC driver - in this case H2 -->
<dependency>
    <!-- Use org.jooq            for the Open Source Edition
             org.jooq.pro        for commercial editions, 
             org.jooq.pro-java-6 for commercial editions with Java 6 support,
             org.jooq.trial      for the free trial edition 
             
         Note: Only the Open Source Edition is hosted on Maven Central. 
               Import the others manually from your distribution -->
    <groupId>org.jooq</groupId>
    <artifactId>jooq</artifactId>
    <version>3.10.8</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.177</version>
</dependency>

<!-- For improved logging, we'll be using log4j via slf4j to see what's going on during migration and code generation -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.16</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.5</version>
</dependency>

<!-- To ensure our code is working, we're using JUnit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
```

***Flyway插件***

```xml
<plugin>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-maven-plugin</artifactId>
    <version>3.0</version>

    <!-- Note that we're executing the Flyway plugin in the "generate-sources" phase -->
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>migrate</goal>
            </goals>
        </execution>
    </executions>

    <!-- Note that we need to prefix the db/migration path with filesystem: to prevent Flyway
         from looking for our migration scripts only on the classpath -->
    <configuration>
        <url>${db.url}</url>
        <user>${db.username}</user>
        <locations>
            <location>filesystem:src/main/resources/db/migration</location>
        </locations>
    </configuration>
</plugin>
```

***jooq的maven配置***

```xml
<plugin>
    <!-- Use org.jooq            for the Open Source Edition
             org.jooq.pro        for commercial editions, 
             org.jooq.pro-java-6 for commercial editions with Java 6 support,
             org.jooq.trial      for the free trial edition 
             
         Note: Only the Open Source Edition is hosted on Maven Central. 
               Import the others manually from your distribution -->
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <version>${org.jooq.version}</version>

    <!-- The jOOQ code generation plugin is also executed in the generate-sources phase, prior to compilation -->
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>

    <!-- This is a minimal working configuration. See the manual's section about the code generator for more details -->
    <configuration>
        <jdbc>
            <url>${db.url}</url>
            <user>${db.username}</user>
        </jdbc>
        <generator>
            <database>
                <includes>.*</includes>
                <inputSchema>FLYWAY_TEST</inputSchema>
            </database>
            <target>
                <packageName>org.jooq.example.flyway.db.h2</packageName>
                <directory>target/generated-sources/jooq-h2</directory>
            </target>
        </generator>
    </configuration>
</plugin>
```

## 数据库增量

当我们开始开发我们的数据库时。为此，我们将创建数据库增量脚本，我们将其放入`src/main/resources/db/migration`目录中，如之前为Flyway插件配置的那样。我们将添加这些文件：

- V1__initialise_database.sql
- V2__create_author_table.sql
- V3__create_book_table_and_records.sql

这三个脚本模拟我们的模式版本1-3（注意大写V！）。这是脚本的内容

```sql
-- V1__initialise_database.sql
DROP SCHEMA flyway_test IF EXISTS;

CREATE SCHEMA flyway_test;
```

```sql
-- V2__create_author_table.sql
CREATE SEQUENCE flyway_test.s_author_id START WITH 1;

CREATE TABLE flyway_test.author (
  id INT NOT NULL,
  first_name VARCHAR(50),
  last_name VARCHAR(50) NOT NULL,
  date_of_birth DATE,
  year_of_birth INT,
  address VARCHAR(50),

  CONSTRAINT pk_author PRIMARY KEY (ID)
);
```

```sql
-- V3__create_book_table_and_records.sql
CREATE TABLE flyway_test.book (
  id INT NOT NULL,
  author_id INT NOT NULL,
  title VARCHAR(400) NOT NULL,

  CONSTRAINT pk_book PRIMARY KEY (id),
  CONSTRAINT fk_book_author_id FOREIGN KEY (author_id) REFERENCES flyway_test.author(id)
);


INSERT INTO flyway_test.author VALUES (next value for flyway_test.s_author_id, 'George', 'Orwell', '1903-06-25', 1903, null);
INSERT INTO flyway_test.author VALUES (next value for flyway_test.s_author_id, 'Paulo', 'Coelho', '1947-08-24', 1947, null);

INSERT INTO flyway_test.book VALUES (1, 1, '1984');
INSERT INTO flyway_test.book VALUES (2, 1, 'Animal Farm');
INSERT INTO flyway_test.book VALUES (3, 2, 'O Alquimista');
INSERT INTO flyway_test.book VALUES (4, 2, 'Brida');
```

## 数据库迁移与代码生成

以上的三个脚本会由Flyway选取并按照版本顺序执行。执行命令

```shell
mvn clean install
```

之后观察log from flyway

```java
[INFO] --- flyway-maven-plugin:3.0:migrate (default) @ jooq-flyway-example ---
[INFO] Database: jdbc:h2:~/flyway-test (H2 1.4)
[INFO] Validated 3 migrations (execution time 00:00.004s)
[INFO] Creating Metadata table: "PUBLIC"."schema_version"
[INFO] Current version of schema "PUBLIC": << Empty Schema >>
[INFO] Migrating schema "PUBLIC" to version 1
[INFO] Migrating schema "PUBLIC" to version 2
[INFO] Migrating schema "PUBLIC" to version 3
[INFO] Successfully applied 3 migrations to schema "PUBLIC" (execution time 00:00.073s).
```

jooq log

```
[INFO] --- jooq-codegen-maven:3.10.8:generate (default) @ jooq-flyway-example ---
[INFO] --- jooq-codegen-maven:3.10.8:generate (default) @ jooq-flyway-example ---
[INFO] Using this configuration:
...
[INFO] Generating schemata      : Total: 1
[INFO] Generating schema        : FlywayTest.java
[INFO] ----------------------------------------------------------
[....]
[INFO] GENERATION FINISHED!     : Total: 337.576ms, +4.299ms
```

## 发展

每当有人向Maven模块添加新的迁移脚本时，所有前面的步骤都会自动执行。例如，团队成员可能已经提交了一个新的迁移脚本，您可以将其检出，重建并获取最新的jOOQ生成的源，以用于您自己的开发或集成测试数据库。

现在，完成这些步骤后，您可以继续编写数据库查询。想象一下以下测试用例

```java
import org.jooq.Result;
import org.jooq.impl.DSL;
import org.junit.Test;

import java.sql.DriverManager;

import static java.util.Arrays.asList;
import static org.jooq.example.flyway.db.h2.Tables.*;
import static org.junit.Assert.assertEquals;

public class AfterMigrationTest {

    @Test
    public void testQueryingAfterMigration() throws Exception {
        try (Connection c = DriverManager.getConnection("jdbc:h2:~/flyway-test", "sa", "")) {
            Result<?> result =
            DSL.using(c)
               .select(
                   AUTHOR.FIRST_NAME,
                   AUTHOR.LAST_NAME,
                   BOOK.ID,
                   BOOK.TITLE
               )
               .from(AUTHOR)
               .join(BOOK)
               .on(AUTHOR.ID.eq(BOOK.AUTHOR_ID))
               .orderBy(BOOK.ID.asc())
               .fetch();

            assertEquals(4, result.size());
            assertEquals(asList(1, 2, 3, 4), result.getValues(BOOK.ID));
        }
    }
}
```

# SQL构建

<https://www.jooq.org/doc/3.10/manual/sql-building/>

## 查询DSL类型

JOOQ暴露了许多接口，并隐藏了客户端代码的大多数实现，原因是

- 接口驱动设计，允许最有效地在流畅的API中建模查询
- 降低客户端代码的复杂性
- API保证，只依赖于公开的接口，而不是具体的实现。

```java
import static org.jooq.impl.DSL.*;
```

***DSL子类***

每种SQL方言都有自己的方言专用DSL，如只使用MySQL方言，则可以选择引用MySQLDSL而不是标准DSL

## DSLContext类

***创建***

```java
// Create it from a pre-existing configuration，从预先的配置中创建
DSLContext create = DSL.using(configuration);

// Create it from ad-hoc arguments，从ad-hoc参数创建
DSLContext create = DSL.using(connection, dialect);
```

***配置***

可以为这些对象提供配置：

- [org.jooq.SQLDialect](https://www.jooq.org/javadoc/3.10.x/org/jooq/SQLDialect.html)：数据库的方言。这可能是任何当前支持的数据库类型（有关详细信息，请参阅[SQL Dialect](https://www.jooq.org/doc/3.10/manual/sql-building/dsl-context/sql-dialects/)）
- [org.jooq.conf.Settings](https://www.jooq.org/javadoc/3.10.x/org/jooq/conf/Settings.html)：可选的运行时配置（有关详细信息，请参阅[自定义设置](https://www.jooq.org/doc/3.10/manual/sql-building/dsl-context/custom-settings/)）
- [org.jooq.ExecuteListenerProvider](https://www.jooq.org/javadoc/3.10.x/org/jooq/ExecuteListenerProvider.html)：对可以为jOOQ提供执行侦听器的提供程序类的可选引用（有关详细信息，请参阅[ExecuteListeners](https://www.jooq.org/doc/3.10/manual/sql-execution/execute-listeners/)）
- [org.jooq.RecordMapperProvider](https://www.jooq.org/javadoc/3.10.x/org/jooq/RecordMapperProvider.html)：对可以为jOOQ提供记录映射器的提供者类的可选引用（有关详细信息，请参阅[带有RecordMappers的POJO](https://www.jooq.org/doc/3.10/manual/sql-execution/fetching/pojos-with-recordmapper-provider/)）
- 任何这些：
  - [java.sql.Connection](https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html)：可选的JDBC连接，将在配置的整个生命周期中重复使用（有关详细信息，请参阅[连接与数据源](https://www.jooq.org/doc/3.10/manual/sql-building/dsl-context/connection-vs-datasource/)）。为简单起见，这是本手册中引用的用例，大部分时间都是如此。
  - [java.sql.DataSource](https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/DataSource.html)：一个可选的JDBC DataSource，它将在Configuration的整个生命周期中重用。如果您更喜欢在Connections上使用DataSources，jOOQ将在内部从您的DataSource获取新的Connections，在查询执行后方便地再次关闭它们。这在J2EE或Spring上下文中特别有用（有关详细信息，请参阅[Connection与DataSource](https://www.jooq.org/doc/3.10/manual/sql-building/dsl-context/connection-vs-datasource/)）
  - [org.jooq.ConnectionProvider](https://www.jooq.org/javadoc/3.10.x/org/jooq/ConnectionProvider.html)：jOOQ用于“获取”和“释放”连接的自定义抽象。jOOQ将在内部“获取”来自ConnectionProvider的新连接，在查询执行后方便地“释放”它们。（有关详细信息，请参阅[Connection vs. DataSource](https://www.jooq.org/doc/3.10/manual/sql-building/dsl-context/connection-vs-datasource/)）

## SQL语句（DML）



## SQL语句（DDL）

# 参考 #

1. 

