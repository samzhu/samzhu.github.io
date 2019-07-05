---
title: Server-Sent Events in Spring
tags:
  - Server-sent events
  - SSE
  - SpringBoot
categories:
  - Spring
---

因為有監控服務是否斷掉的需求, 要做一個 Dashboard 來即時呈現問題點,  
但用輪詢怕會自己把自己搞死, 所以就看了一下 Server-sent events,  
也不是什麼新東西了, 主流瀏覽器也都支援了, 大概沒什麼或坑吧 XD  

就拿來試玩一下順便紀錄.

<!--more-->

好的, 首先先看看你專案是用 web 還是 webflux?  
如果是 web 看這邊 [Async Requests - HTTP Streaming - SSE](https://docs.spring.io/spring/docs/5.0.x/spring-framework-reference/web.html#mvc-ann-async-sse)  
如果是 webFlux 的看這邊 [Reactive Core - Codecs - HTTP Streaming](https://docs.spring.io/spring/docs/5.0.x/spring-framework-reference/web-reactive.html#webflux-codecs-streaming)  

我這邊是用 webflux 練習, 就沒有練習 SseEmitter 的用法囉  

## Dependency
pom.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

## Code
我準備兩個 VO 物件給頁面呈現使用, 這 VO 很簡單就是裝載現在時間跟服務的名稱.  

ClusterHealthVo.java
``` java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ClusterHealthVo {
    private List<ApplicationVo> clusterHealth;
    private Instant nowTime;
}
```

ApplicationVo.java
``` java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ApplicationVo {
    private String name;
}
```

我用了一個模擬的服務列表  

ApplicationService.java
``` java
@Service
public class ApplicationService {
    private static List<ApplicationVo> applicationVoList = new ArrayList<>();
    // 新增服務
    public void addApplication(ApplicationVo applicationVo){
        applicationVoList.add(applicationVo);
    }
    // 取得服務清單
    public List<ApplicationVo> getApplicationVoList(){
        return applicationVoList;
    }
}
```

再加上一個定時器模擬服務增加的動作  

ApplicationLoader.java
``` java
@Slf4j
@Component
@RequiredArgsConstructor
public class ApplicationLoader {
    private final ApplicationService applicationService;
    private final AtomicInteger atomicInteger = new AtomicInteger();

    @EventListener(ApplicationReadyEvent.class)
    public void applicationReady() {
    }

    @Scheduled(fixedRate = 5000)
    public void schedule() {
        applicationService.addApplication(new ApplicationVo("Sam_" + atomicInteger.incrementAndGet()));
    }
}
```

使用到 Scheduled 別忘了啟動  

DemoApplication.java  
``` java
@EnableScheduling
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

都好了之後這邊就是後端 SSE 的實做啦  
SseController.java
``` java
@RestController
@RequestMapping("/sse")
@RequiredArgsConstructor
public class SseController {
    private final ApplicationService applicationService;

    @GetMapping(path = "/stream-flux", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ClusterHealthVo> streamFlux() {
        return Flux.interval(Duration.ofSeconds(1))
                .map(sequence -> new ClusterHealthVo(applicationService.getApplicationVoList(), Instant.now()));
    }

    @GetMapping("/stream-sse")
    public Flux<ServerSentEvent<ClusterHealthVo>> streamEvents() {
        return Flux.interval(Duration.ofSeconds(1))
                .map(sequence -> ServerSentEvent.<ClusterHealthVo>builder()
                        .id(String.valueOf(sequence))
                        .event("health-check-event")
                        .data(new ClusterHealthVo(applicationService.getApplicationVoList(), Instant.now()))
                        .build());
    }
}
```

其實功能是一樣的, 差別是有沒有自訂 eventType  

如果我們分別用瀏覽器直接打開 API 的網址會看到他一直從後端輸出資料  

http://localhost/sse/stream-flux
``` text
data:{"clusterHealth":[{"name":"Sam_1"}],"nowTime":"2019-07-05T08:49:36.890Z"}

data:{"clusterHealth":[{"name":"Sam_1"}],"nowTime":"2019-07-05T08:49:37.890Z"}

data:{"clusterHealth":[{"name":"Sam_1"}],"nowTime":"2019-07-05T08:49:38.890Z"}

data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"}],"nowTime":"2019-07-05T08:49:39.890Z"}

data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"}],"nowTime":"2019-07-05T08:49:40.890Z"}

data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"}],"nowTime":"2019-07-05T08:49:41.889Z"}

data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"}],"nowTime":"2019-07-05T08:49:42.889Z"}

data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"}],"nowTime":"2019-07-05T08:49:43.890Z"}

data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"},{"name":"Sam_3"}],"nowTime":"2019-07-05T08:49:44.889Z"}

data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"},{"name":"Sam_3"}],"nowTime":"2019-07-05T08:49:45.889Z"}

data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"},{"name":"Sam_3"}],"nowTime":"2019-07-05T08:49:46.889Z"}
```

使用 ServerSentEvent 則是會輸出這樣的資料  

http://localhost/sse/stream-sse
``` text
id:0
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"}],"nowTime":"2019-07-05T08:51:20.442Z"}

id:1
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"}],"nowTime":"2019-07-05T08:51:21.441Z"}

id:2
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"}],"nowTime":"2019-07-05T08:51:22.441Z"}

id:3
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"}],"nowTime":"2019-07-05T08:51:23.441Z"}

id:4
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"}],"nowTime":"2019-07-05T08:51:24.441Z"}

id:5
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"}],"nowTime":"2019-07-05T08:51:25.441Z"}

id:6
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"}],"nowTime":"2019-07-05T08:51:26.443Z"}

id:7
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"}],"nowTime":"2019-07-05T08:51:27.442Z"}

id:8
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"},{"name":"Sam_3"}],"nowTime":"2019-07-05T08:51:28.441Z"}

id:9
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"},{"name":"Sam_3"}],"nowTime":"2019-07-05T08:51:29.441Z"}

id:10
event:health-check-event
data:{"clusterHealth":[{"name":"Sam_1"},{"name":"Sam_2"},{"name":"Sam_3"}],"nowTime":"2019-07-05T08:51:30.442Z"}
```


來看看前端網頁怎麼些收資料  

demo-flux.html
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Server-Sent Events Demo</title>
</head>
<body>
<h1>Server-Sent Events Demo - Flux&lt;ClusterHealthVo&gt;</h1>
<div id="vue-display">
    <table>
        <tr v-for="item in clusterHealth">
            <td>{{ item.name }}</td>
        </tr>
    </table>
    <div>刷新時間</div>
    <div>{{  nowTime }}</div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.6.10/vue.min.js"></script>
<script>
    Vue.config.devtools = true;
    Vue.config.debug = true;

    window.addEventListener('load', function (ev) {
        var es = new EventSource('/sse/stream-flux')
        es.addEventListener('message', function (msg) {
            console.log(msg.data);
            var reObj = JSON.parse(msg.data);
            vueMainContent.clusterHealth = reObj.clusterHealth;
            vueMainContent.nowTime = reObj.nowTime;
        });
        es.addEventListener("open", function (e) {
            console.log('連接成功');
        });
        es.addEventListener("error", function (e) {
            console.log('網路異常 最後連線時間<br/>' + vueMainContent.nowTime);
            vueMainContent.clusterHealth = [];
        });
    });

    var vueMainContent = new Vue({
        el: '#vue-display',
        data: {
            clusterHealth: [],
            nowTime: ''
        }
    });
</script>
</body>
</html>
```

如果是接 Flux<ServerSentEvent<ClusterHealthVo>> 的話要對應一下 event 的名稱  
demo-sse.html
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Server-Sent Events Demo</title>
</head>
<body>
<h1>Server-Sent Events Demo - Flux&lt;ServerSentEvent&lt;ClusterHealthVo&gt;&gt;</h1>
<div id="vue-display">
    <table>
        <tr v-for="item in clusterHealth">
            <td>{{ item.name }}</td>
        </tr>
    </table>
    <div>刷新時間</div>
    <div>{{  nowTime }}</div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.6.10/vue.min.js"></script>
<script>
    Vue.config.devtools = true;
    Vue.config.debug = true;

    window.addEventListener('load', function (ev) {
        var es = new EventSource('/sse/stream-sse')
        es.addEventListener('message', function (msg) {
            // 有自訂就不會走 message 這個
            console.log("message");
        });
        es.addEventListener("open", function (e) {
            console.log('連接成功');
        });
        es.addEventListener("error", function (e) {
            console.log('網路異常 最後連線時間<br/>' + vueMainContent.nowTime);
        });
        es.addEventListener('health-check-event', function(msg) {
            console.log('is health-check-event!!');
            var reObj = JSON.parse(msg.data);
            vueMainContent.clusterHealth = reObj.clusterHealth;
            vueMainContent.nowTime = reObj.nowTime;
        }, false);
    });

    var vueMainContent = new Vue({
        el: '#vue-display',
        data: {
            clusterHealth: [],
            nowTime: ''
        }
    });
</script>
</body>
</html>
```

差別是不會統一走 message 接收進來
``` js
es.addEventListener('message', function (msg) {
  // 有自訂就不會走 message 這個
  console.log("message");
});
```

## References
[Server-Sent Events in Spring - baeldung](https://www.baeldung.com/spring-server-sent-events)  
[Async Requests - HTTP Streaming - SSE - Spring Framework Documentation](https://docs.spring.io/spring/docs/5.0.x/spring-framework-reference/web.html#mvc-ann-async-sse)  
[Reactive Core - Codecs - HTTP Streaming - Spring Framework Documentation](https://docs.spring.io/spring/docs/5.0.x/spring-framework-reference/web-reactive.html#webflux-codecs-streaming)  
[Server-Sent Events でチャットアプリを作ってみた (Spring Boot 2.x系 x Spring MVC x Akka Actor)](https://techblog.securesky-tech.com/entry/2018/12/28/)  
[Server-Sent Events with Spring](https://golb.hplar.ch/2017/03/Server-Sent-Events-with-Spring.html)  
[Spring 5 新增全新的reactive web框架：webflux](https://cloud.tencent.com/developer/article/1082833)  
[使用server side event 實作聊天功能](https://tpu.thinkpower.com.tw/tpu/articleDetails/721)  
[使用 Spring 5 的 Webflux 开发 Reactive 应用](https://windmt.com/2018/04/25/spring-web-reactive-webflux/)  

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。