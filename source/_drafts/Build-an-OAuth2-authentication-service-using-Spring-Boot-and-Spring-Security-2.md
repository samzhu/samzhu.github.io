---
title: >-
  Build an OAuth2 authentication service using Spring Boot and Spring
  Security(2)
tags:
  - SpringBoot
  - SpringSecurity
  - Oauth
categories:
  - Spring
---

第二篇 也是很簡單 先把帳號密碼放在記憶體吧  

配置有兩種  

<!--more-->

## yml
application-dev.yml
``` yml
spring:
  security:
    user:
      name: 'sam'
      password: '1qaz2wsx'
      roles: 'ADMIN'
```
這樣就可以啟用 Basic Auth  

測試  
``` bash
curl -X GET \
  http://localhost:8080 \
  -H 'Authorization: Basic c2FtOjFxYXoyd3N4' 
```
這樣就可以拿到回應了.

## JavaConfiguration

可以做更多細部調整  

ServerSecurityConfiguration.java  
``` java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@EnableWebSecurity
public class MyWebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("sam").password("{noop}1qaz2wsx").roles("USER")
                .and()
                .withUser("admin").password("{noop}password").roles("USER", "ADMIN");
        // 測試用 {noop} 表示明碼儲存
    }

    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest()
                .authenticated()
                .and()
                .httpBasic();
    }
    
//    真的要用時需要用 Encoder 驗證密碼
//    @Bean
//    public PasswordEncoder passwordEncoder() {
//        return new BCryptPasswordEncoder();
//    }
}
```

測試  
``` bash
curl -X GET \
  http://localhost:8080 \
  -H 'Authorization: Basic c2FtOjFxYXoyd3N4' 
```
一樣可以拿到資料

## References
[Spring REST + Spring Security Example](https://www.mkyong.com/spring-boot/spring-rest-spring-security-example/)  

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。