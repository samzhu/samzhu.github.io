---
title: SpringBoot 2.4 with GraalVM
date: 2020-12-13 22:34:15
tags:
  - SpringBoot
  - GraalVM
categories:
  - Spring
---

最近很紅的 Quarkus 也是使用 GraalVM, 並盡量避免使用 反射(Reflection) 達成 快數啟動 低記憶體用量等優點, 等了許久終於等到 SpringBoot 2.4 終於支援了 GraalVM, 所以趕快試用一下

## create project
首先還是透過 https://start.spring.io/ 下載專案模板, 目前是 2.4 並且選擇 Maven 包版工具(Gradle Plugin 還在測試版就不折騰啦)
```
curl https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.4.1.RELEASE&packaging=jar&jvmVersion=11&groupId=com.example&artifactId=demo&name=demo&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.demo&dependencies=web,actuator
```

## 程式碼
程式部分先做最簡單的驗證  
``` java
public class Product {
    private String name;
    private Integer price;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Integer getPrice() {
        return price;
    }
    public void setPrice(Integer price) {
        this.price = price;
    }
}
```

跟簡單的 API
``` java
@RestController
public class ProductController {

    @PostMapping("/products")
    public Product create(@RequestBody Product product) {
        return product;
    }

    @GetMapping("/products")
    public List getProducts() {
        List productList = IntStream.range(0, 7).mapToObj(i -> getProduct(i)).collect(Collectors.toList());
        return productList;
    }

    private Product getProduct(int i) {
        Product product = new Product();
        product.setName("Product" + i);
        product.setPrice(100 + i);
        return product;
    }
}
```

## 打包運行
``` bash
./mvnw package
```

到這邊都是最簡單的 API 功能, 接著啟動 Jar 觀測一下

``` bash
$ java -jar ./target/demo-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.4.1)

2020-12-13 23:28:22.327  INFO 23045 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication v0.0.1-SNAPSHOT using Java 11.0.9 on samzhude-MacBook-Pro.local with PID 23045 (/Users/samzhu/workspace/demo/springboot-241-GraalVM/target/demo-0.0.1-SNAPSHOT.jar started by samzhu in /Users/samzhu/workspace/demo/springboot-241-GraalVM)
2020-12-13 23:28:22.330  INFO 23045 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
2020-12-13 23:28:23.398  INFO 23045 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-12-13 23:28:23.413  INFO 23045 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-12-13 23:28:23.414  INFO 23045 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.41]
2020-12-13 23:28:23.477  INFO 23045 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-12-13 23:28:23.478  INFO 23045 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1085 ms
2020-12-13 23:28:23.790  INFO 23045 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-12-13 23:28:24.013  INFO 23045 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 2 endpoint(s) beneath base path '/actuator'
2020-12-13 23:28:24.066  INFO 23045 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-12-13 23:28:24.080  INFO 23045 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 2.212 seconds (JVM running for 2.647)
```
就算是這麼簡單的 API 功能, 啟動都需要 2 秒多, 更不用說完整的應用 DataSource 以及許多初始化, 以及更受限的資源 ex 0.5 cpu 之類的, 啟動時間超過 30 秒更是家常便飯.  
  
所以線上環境 Scale Out 的時候, 我都彷彿聽見新起動的 SpringBoot 在這樣吶喊
![幫我撐10秒](https://i.imgur.com/qiCljKi.png)
  
看一下記憶體用量  
``` bash
$ ps aux | grep java | awk '{print $1 "\t" $2 "\t" $3 "\t" $4 "\t" $5 "\t" $6/1024"MB" "\t" $11$12$13$14$15$16$17$18 }'
samzhu	21097	0.3	1.6	7385948	257.922MB	/Users/samzhu/.sdkman/candidates/java/current/bin/java--add-modules=ALL-SYSTEM--add-opensjava.base/java.util=ALL-UNNAMED--add-opensjava.base/java.lang=ALL-UNNAMED-Declipse.application=org.eclipse.jdt.ls.core.id1-Dosgi.bundles.defaultStartLevel=4
samzhu	21104	0.0	0.7	10636212	115.102MB	/Users/samzhu/.sdkman/candidates/java/11.0.9.hs-adpt/bin/java-cp/Users/samzhu/.vscode/extensions/pivotal.vscode-spring-boot-1.23.0/language-server/BOOT-INF/classes:/Users/samzhu/.vscode/extensions/pivotal.vscode-spring-boot-1.23.0/language-server/BOOT-INF/lib/*-Dspring.lsp.client-port=45557-Dserver.port=45557-Dsts.lsp.client=vscode-Dsts.log.file=/dev/null-XX:TieredStopAtLevel=1
samzhu	20995	0.0	1.1	10639344	174.391MB	/Users/samzhu/.sdkman/candidates/java/11.0.9.hs-adpt/bin/java-cp/Users/samzhu/.vscode/extensions/pivotal.vscode-spring-boot-1.23.0/language-server/BOOT-INF/classes:/Users/samzhu/.vscode/extensions/pivotal.vscode-spring-boot-1.23.0/language-server/BOOT-INF/lib/*-Dspring.lsp.client-port=45556-Dserver.port=45556-Dsts.lsp.client=vscode-Dsts.log.file=/dev/null-XX:TieredStopAtLevel=1
samzhu	20278	0.0	4.4	7513840	717.336MB	/Users/samzhu/.sdkman/candidates/java/current/bin/java--add-modules=ALL-SYSTEM--add-opensjava.base/java.util=ALL-UNNAMED--add-opensjava.base/java.lang=ALL-UNNAMED-Declipse.application=org.eclipse.jdt.ls.core.id1-Dosgi.bundles.defaultStartLevel=4
samzhu	8875	0.0	2.4	7450684	390.492MB	/Users/samzhu/.sdkman/candidates/java/current/bin/java--add-modules=ALL-SYSTEM--add-opensjava.base/java.util=ALL-UNNAMED--add-opensjava.base/java.lang=ALL-UNNAMED-Declipse.application=org.eclipse.jdt.ls.core.id1-Dosgi.bundles.defaultStartLevel=4
samzhu	24462	0.0	0.0	4268424	0.679688MB	grepjava
samzhu	24079	0.0	1.3	10381056	209.754MB	java-jar./target/demo-0.0.1-SNAPSHOT.jar
```
看起來隨隨便便都要 209.754MB 啊...平常在用的幾乎都要 512 MB 才比較穩定  

好的, 那既然 SpringBoot 支援 GraalVM 了, 我們何不感受一下他的威力呢

## 改成使用GraalVM包版



