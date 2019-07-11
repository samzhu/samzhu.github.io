---
title: >-
  Build an OAuth2 authentication service using Spring Boot and Spring
  Security(1)
tags:
  - SpringBoot
  - SpringSecurity
  - Oauth
categories:
  - Spring
date: 2019-07-11 16:57:08
---


又要做 OAuth 服務啦!! 那為什麼是 (1) 呢? 因為這次會分好幾版來記錄, 這樣之後我自己在找比較好找從哪一版適合挑出來改或教學 XD.

久久要重用以前的專案時, 習慣就會把專案再整理整理, 把當時沒考慮到的做好, 這樣不知道是不是壞習慣, 常常要花時間在整理.

<!--more-->

這次用的是 SpringBoot 2.1.6 版  

依賴如下  
build.gradle
``` groovy
plugins {
    id 'org.springframework.boot' version '2.1.6.RELEASE'
    id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.benwon'
version = '0.0.1'
sourceCompatibility = '1.8'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

ext {
    set('springCloudVersion', "Greenwich.SR1")
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-data-rest'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.flywaydb:flyway-core'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'mysql:mysql-connector-java'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}
```

基本上加上 org.springframework.boot:spring-boot-starter-security 後就有基本的配置啦, 所以我們可以直接啟動  
``` console
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.6.RELEASE)

2019-07-11 15:36:48.168  INFO 9108 --- [           main] com.benwon.auth.AuthApplication          : Starting AuthApplication on FD100112 with PID 9108 (D:\IdeaProjects\sam\auth\build\classes\java\main started by Sam.Chu in D:\IdeaProjects\sam\auth)
2019-07-11 15:36:48.170  INFO 9108 --- [           main] com.benwon.auth.AuthApplication          : The following profiles are active: dev
2019-07-11 15:36:48.695  INFO 9108 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data repositories in DEFAULT mode.
2019-07-11 15:36:48.707  INFO 9108 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 7ms. Found 0 repository interfaces.
2019-07-11 15:36:48.948  INFO 9108 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration$$EnhancerBySpringCGLIB$$6cf0c6b9] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2019-07-11 15:36:48.962  INFO 9108 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.hateoas.config.HateoasConfiguration' of type [org.springframework.hateoas.config.HateoasConfiguration$$EnhancerBySpringCGLIB$$ec7113eb] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2019-07-11 15:36:49.157  INFO 9108 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2019-07-11 15:36:49.169  INFO 9108 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-07-11 15:36:49.169  INFO 9108 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.21]
2019-07-11 15:36:49.237  INFO 9108 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-07-11 15:36:49.237  INFO 9108 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1045 ms
2019-07-11 15:36:49.785  INFO 9108 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2019-07-11 15:36:49.951  INFO 9108 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2019-07-11 15:36:49.987  INFO 9108 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [
	name: default
	...]
2019-07-11 15:36:50.022  INFO 9108 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {5.3.10.Final}
2019-07-11 15:36:50.022  INFO 9108 --- [           main] org.hibernate.cfg.Environment            : HHH000206: hibernate.properties not found
2019-07-11 15:36:50.099  INFO 9108 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.0.4.Final}
2019-07-11 15:36:50.176  INFO 9108 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.MySQL5Dialect
2019-07-11 15:36:50.619  INFO 9108 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2019-07-11 15:36:51.026  INFO 9108 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-07-11 15:36:51.054  WARN 9108 --- [           main] aWebConfiguration$JpaWebMvcConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2019-07-11 15:36:51.398  INFO 9108 --- [           main] .s.s.UserDetailsServiceAutoConfiguration : 

Using generated security password: d8175a33-911e-4047-ac8a-d9322c0561dd

2019-07-11 15:36:51.495  INFO 9108 --- [           main] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: any request, [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@a0e33db, org.springframework.security.web.context.SecurityContextPersistenceFilter@112530c3, org.springframework.security.web.header.HeaderWriterFilter@b76b7d8, org.springframework.security.web.csrf.CsrfFilter@68f2363, org.springframework.security.web.authentication.logout.LogoutFilter@4cadd4d4, org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@2890e479, org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@4c28f97e, org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@3ef46749, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@2a984952, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@355493bf, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@4e647f39, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@40b54762, org.springframework.security.web.session.SessionManagementFilter@63c4d16, org.springframework.security.web.access.ExceptionTranslationFilter@702cfbde, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@5600a278]
2019-07-11 15:36:51.575  INFO 9108 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 2 endpoint(s) beneath base path '/actuator'
2019-07-11 15:36:51.664  INFO 9108 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-07-11 15:36:51.665  INFO 9108 --- [           main] com.benwon.auth.AuthApplication          : Started AuthApplication in 3.722 seconds (JVM running for 4.136)
2019-07-11 15:36:51.778  INFO 9108 --- [2)-192.168.56.1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2019-07-11 15:36:51.778  INFO 9108 --- [2)-192.168.56.1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2019-07-11 15:36:51.892  INFO 9108 --- [2)-192.168.56.1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 114 ms
```

你會看到畫面有印一個訊息
```
Using generated security password: d8175a33-911e-4047-ac8a-d9322c0561dd
```

這就是隨機產生的密碼啦  

接著打開網頁 [http://localhost:8080/login](http://localhost:8080/login)  
![https://i.imgur.com/p9opLBl.png](https://i.imgur.com/p9opLBl.png)  
就在 網頁上輸入  
帳號: user  
密碼: d8175a33-911e-4047-ac8a-d9322c0561dd  

就可以登入啦


## References
[Hello Spring Security with Boot](https://docs.spring.io/spring-security/site/docs/current/guides/html5/helloworld-boot.html)  

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。