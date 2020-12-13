---
title: 方便管理JDK版本工具sdkman
date: 2020-12-13 21:05:22
tags:
  - sdkman
categories:
  - Java
---

有時還是有不同的專案對應不同的 JDK 版本, 雖然你可以全都裝, 但管理還是麻煩, 所以裝了這個 JDK 管理工具, 方便做切換

<!--more-->

## install sdkman

官網在這邊 (sdkman)[https://sdkman.io/install]  

安裝很簡單, 一行指令搞定
``` bash
curl -s "https://get.sdkman.io" | bash
```

## how to use

使用指令, 可以列出所有支援的 Java 版本, 包含 AdoptOpenJDK Amazon Azul Zulu 可說是該有的都有了, 居然還有 GraalVM
``` bash
$ sdk list java
================================================================================
Available Java Versions
================================================================================
 Vendor        | Use | Version      | Dist    | Status     | Identifier
--------------------------------------------------------------------------------
 AdoptOpenJDK  |     | 15.0.1.j9    | adpt    |            | 15.0.1.j9-adpt      
               |     | 15.0.1.hs    | adpt    |            | 15.0.1.hs-adpt      
               |     | 14.0.2.j9    | adpt    |            | 14.0.2.j9-adpt      
               |     | 14.0.2.hs    | adpt    |            | 14.0.2.hs-adpt      
               |     | 13.0.2.j9    | adpt    |            | 13.0.2.j9-adpt      
               |     | 13.0.2.hs    | adpt    |            | 13.0.2.hs-adpt      
               |     | 12.0.2.j9    | adpt    |            | 12.0.2.j9-adpt      
               |     | 12.0.2.hs    | adpt    |            | 12.0.2.hs-adpt      
               |     | 11.0.9.j9    | adpt    |            | 11.0.9.j9-adpt      
               | >>> | 11.0.9.hs    | adpt    | installed  | 11.0.9.hs-adpt      
               |     | 8.0.275.j9   | adpt    |            | 8.0.275.j9-adpt     
               |     | 8.0.275.hs   | adpt    |            | 8.0.275.hs-adpt     
               |     | 8.0.272.hs   | adpt    |            | 8.0.272.hs-adpt     
 Amazon        |     | 15.0.1       | amzn    |            | 15.0.1-amzn         
               |     | 11.0.9       | amzn    |            | 11.0.9-amzn         
               |     | 8.0.275      | amzn    |            | 8.0.275-amzn        
 Azul Zulu     |     | 15.0.1       | zulu    |            | 15.0.1-zulu         
               |     | 15.0.1.fx    | zulu    |            | 15.0.1.fx-zulu      
               |     | 14.0.2       | zulu    |            | 14.0.2-zulu         
               |     | 14.0.2.fx    | zulu    |            | 14.0.2.fx-zulu      
               |     | 13.0.5       | zulu    |            | 13.0.5-zulu         
               |     | 13.0.5.fx    | zulu    |            | 13.0.5.fx-zulu      
               |     | 12.0.2       | zulu    |            | 12.0.2-zulu         
               |     | 11.0.9       | zulu    |            | 11.0.9-zulu         
               |     | 11.0.9.fx    | zulu    |            | 11.0.9.fx-zulu      
               |     | 10.0.2       | zulu    |            | 10.0.2-zulu         
               |     | 9.0.7        | zulu    |            | 9.0.7-zulu          
               |     | 8.0.272      | zulu    |            | 8.0.272-zulu        
               |     | 8.0.272.fx   | zulu    |            | 8.0.272.fx-zulu     
               |     | 8.0.202      | zulu    |            | 8.0.202-zulu        
               |     | 7.0.282      | zulu    |            | 7.0.282-zulu        
               |     | 7.0.181      | zulu    |            | 7.0.181-zulu        
 BellSoft      |     | 15.0.1.fx    | librca  |            | 15.0.1.fx-librca    
               |     | 15.0.1       | librca  |            | 15.0.1-librca       
               |     | 14.0.2.fx    | librca  |            | 14.0.2.fx-librca    
               |     | 14.0.2       | librca  |            | 14.0.2-librca       
               |     | 13.0.2.fx    | librca  |            | 13.0.2.fx-librca    
               |     | 13.0.2       | librca  |            | 13.0.2-librca       
               |     | 12.0.2       | librca  |            | 12.0.2-librca       
               |     | 11.0.9.fx    | librca  |            | 11.0.9.fx-librca    
               |     | 11.0.9       | librca  |            | 11.0.9-librca       
               |     | 8.0.275.fx   | librca  |            | 8.0.275.fx-librca   
               |     | 8.0.275      | librca  |            | 8.0.275-librca      
               |     | 8.0.265.fx   | librca  |            | 8.0.265.fx-librca   
 GraalVM       |     | 20.3.0.r11   | grl     | installed  | 20.3.0.r11-grl      
               |     | 20.3.0.r8    | grl     | installed  | 20.3.0.r8-grl       
               |     | 20.2.0.r11   | grl     | installed  | 20.2.0.r11-grl      
               |     | 20.2.0.r8    | grl     |            | 20.2.0.r8-grl       
               |     | 20.1.0.r11   | grl     |            | 20.1.0.r11-grl      
               |     | 20.1.0.r8    | grl     |            | 20.1.0.r8-grl       
               |     | 20.0.0.r11   | grl     |            | 20.0.0.r11-grl      
               |     | 20.0.0.r8    | grl     |            | 20.0.0.r8-grl       
               |     | 19.3.4.r11   | grl     |            | 19.3.4.r11-grl      
               |     | 19.3.4.r8    | grl     |            | 19.3.4.r8-grl       
               |     | 19.3.1.r11   | grl     |            | 19.3.1.r11-grl      
               |     | 19.3.1.r8    | grl     |            | 19.3.1.r8-grl       
 Java.net      |     | 16.ea.28     | open    |            | 16.ea.28-open       
               |     | 16.ea.9.lm   | open    |            | 16.ea.9.lm-open     
               |     | 16.ea.3.pma  | open    |            | 16.ea.3.pma-open    
               |     | 15.0.1       | open    |            | 15.0.1-open         
               |     | 14.0.2       | open    |            | 14.0.2-open         
               |     | 13.0.2       | open    |            | 13.0.2-open         
               |     | 12.0.2       | open    |            | 12.0.2-open         
               |     | 11.0.2       | open    |            | 11.0.2-open         
               |     | 10.0.2       | open    |            | 10.0.2-open         
               |     | 9.0.4        | open    |            | 9.0.4-open          
 SAP           |     | 15.0.1       | sapmchn |            | 15.0.1-sapmchn      
               |     | 14.0.2       | sapmchn |            | 14.0.2-sapmchn      
               |     | 13.0.2       | sapmchn |            | 13.0.2-sapmchn      
               |     | 12.0.2       | sapmchn |            | 12.0.2-sapmchn      
               |     | 11.0.9       | sapmchn |            | 11.0.9-sapmchn      
 TravaOpenJDK  |     | 11.0.9       | trava   |            | 11.0.9-trava        
               |     | 8.0.232      | trava   |            | 8.0.232-trava       
================================================================================
Use the Identifier for installation:

    $ sdk install java 11.0.3.hs-adpt
================================================================================
```

安裝指定版本
``` bash
$ sdk install java 11.0.9.hs-adpt

Found a previously downloaded java 11.0.9.hs-adpt archive. Not downloading it again...

Installing: java 11.0.9.hs-adpt
Done installing!

Do you want java 11.0.9.hs-adpt to be set as default? (Y/n): y

Setting java 11.0.9.hs-adpt as default.
```

移除指定版本
``` bash
$ sdk uninstall java 11.0.9.hs-adpt

Deselecting java 11.0.9.hs-adpt...

Uninstalling java 11.0.9.hs-adpt...
```

雖然說有多版本要用在切換是很方便, 但還是會有預設版本的需求
```
$ sdk default java 11.0.9.hs-adpt

Default java version set to 11.0.9.hs-adpt
```

這時候去驗證一下
``` bash
$ java -version
openjdk version "11.0.9" 2020-10-20
OpenJDK Runtime Environment AdoptOpenJDK (build 11.0.9+11)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 11.0.9+11, mixed mode)
```

輕鬆管理不費力, 讚!

