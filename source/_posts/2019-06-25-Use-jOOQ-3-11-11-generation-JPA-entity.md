---
title: Use jOOQ 3.11.11 generation JPA entity
tags:
  - jooq
categories:
  - Java
date: 2019-06-25 14:58:56
---


自從 Oracle JDK 喊說要收費後, 就陸續將開發機換成 OpenJDK,  
但是以 Windows 來說 Amazon Corretto 提供的最方便安裝了,  
但換成 Corretto 11 後, 有缺一些依賴所以又要改一下指令所以紀錄一下.  

<!--more-->

``` bash
PS D:\jOOQ-3.11.11> java -version
openjdk version "11.0.2" 2019-01-15 LTS
OpenJDK Runtime Environment Corretto-11.0.2.9.1 (build 11.0.2+9-LTS)
OpenJDK 64-Bit Server VM Corretto-11.0.2.9.1 (build 11.0.2+9-LTS, mixed mode)
```

library.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration xmlns="http://www.jooq.org/xsd/jooq-codegen-3.11.0.xsd">
   <!-- Configure the database connection here -->
   <jdbc>
      <driver>org.postgresql.Driver</driver>
      <url>jdbc:postgresql://127.0.0.1/test</url>
      <user>postgres</user>
      <password>1qaz2wsx</password>
   </jdbc>
   <generator>
      <name>org.jooq.codegen.JavaGenerator</name>
      <database>
         <name>org.jooq.meta.postgres.PostgresDatabase</name>
         <inputSchema>public</inputSchema>
         <includes>.*</includes>
         <excludes />
      </database>
      <target>
         <packageName>com.benwon.admin.entity</packageName>
         <directory>D:\jOOQ-3.11.11\benwon\jooq\src\main\java</directory>
      </target>
      <generate>
         <records>false</records>
         <pojos>true</pojos>
         <jpaAnnotations>true</jpaAnnotations>
         <jpaVersion>2.2</jpaVersion>
         <validationAnnotations>true</validationAnnotations>
         <springAnnotations>true</springAnnotations>
         <pojosToString>false</pojosToString>
         <javaTimeTypes>true</javaTimeTypes>
         <!-- 使用 Java 8 的 LocalDateTime -->
      </generate>
   </generator>
</configuration>
```

jooq 下載  
[jOOQ](https://www.jooq.org/download/)  

需要額外的依賴  
[JAXB API » 2.3.1](https://mvnrepository.com/artifact/javax.xml.bind/jaxb-api/2.3.1)  
[Old JAXB Runtime » 2.3.2](https://mvnrepository.com/artifact/com.sun.xml.bind/jaxb-impl/2.3.2)  
[JavaBeans(TM) Activation Framework » 1.1.1](https://mvnrepository.com/artifact/javax.activation/activation/1.1.1)  
[JAXB API » 2.3.1](https://mvnrepository.com/artifact/javax.xml.bind/jaxb-api/2.3.1)  
[PostgreSQL JDBC Driver JDBC 4.2 » 42.2.5](https://mvnrepository.com/artifact/org.postgresql/postgresql/42.2.5)  

Windows PowerShell
``` bash
PS D:\jOOQ-3.11.11> java -classpath "jooq-3.11.11.jar;jooq-meta-3.11.11.jar;jooq-codegen-3.11.11.jar;jaxb-api-2.3.1.jar;jaxb-impl-2.3.2.jar;activation-1.1.1.jar;postgresql-42.2.5.jar" org.jooq.codegen.GenerationTool library.xml
6月 25, 2019 2:55:15 下午 org.jooq.tools.JooqLogger info
資訊: Initialising properties  : library.xml
6月 25, 2019 2:55:15 下午 org.jooq.tools.JooqLogger info
資訊: No <inputCatalog/> was provided. Generating ALL available catalogs instead.
6月 25, 2019 2:55:15 下午 org.jooq.tools.JooqLogger info
資訊: License parameters
.....略
6月 25, 2019 2:55:16 下午 org.jooq.tools.JooqLogger info
資訊: Indexes generated        : Total: 622.778ms, +1.818ms
6月 25, 2019 2:55:16 下午 org.jooq.tools.JooqLogger info
資訊: Domains fetched          : 0 (0 included, 0 excluded)
6月 25, 2019 2:55:16 下午 org.jooq.tools.JooqLogger info
資訊: Generation finished: public: Total: 635.468ms, +12.689ms
6月 25, 2019 2:55:16 下午 org.jooq.tools.JooqLogger info
資訊:
6月 25, 2019 2:55:16 下午 org.jooq.tools.JooqLogger info
資訊: Removing excess files
```

完成

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。