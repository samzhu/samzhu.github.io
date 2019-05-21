---
title: 註冊 GitLab Runner use Docker
tags:
  - GitLab
  - Docker
categories:
  - DevOps
date: 2019-05-02 14:33:00
---

程式寫完, 接著就是要打包輸出了  

這個 CI 階段, 我是選擇用 GitLab Runner ( Docker in Docker 方案 )

<!--more-->
首先用 root 身分 登入後  
打開 settings -> CI / CD  
展開 Runners  

你會看到類似下面畫面
![Imgur](https://i.imgur.com/EZneRW8.png)

這邊你可以得到 Runner 註冊時需要的 url 跟 token 先把他們記錄下來  
接下來  
我們進入到要當 Runner 的主機中執行
``` bash
[root@cs-gitlab-runner ~]# docker run --rm -t -i -v /path/to/config:/etc/gitlab-runner \
--name gitlab-runner \
gitlab/gitlab-runner register \
>   --non-interactive \
>   --executor "docker" \
>   --docker-image alpine:3 \
>   --url "http://gitlab.xxxxx.com/" \
>   --registration-token "FsuxxxxxxxxdVF4" \
>   --description "docker-runner" \
>   --run-untagged \
>   --locked="false"
Runtime platform                                    arch=amd64 os=linux pid=6 revision=8bb608ff version=11.7.0
Running in system-mode.

Registering runner... succeeded                     runner=FsuUn-ca
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

之後你就可以看到 Runner 有沒有註冊成功,  
但這個時候應該是紅燈, 表示已註冊但不能用, 要讓他執行起來就會向下圖一樣變綠燈了
![Imgur](https://i.imgur.com/EQ7rmsu.png)

然後我們先看一下 /path/to/config/config.toml 這個檔案  
這檔案是 register 時候會自動產生的  
```
[root@cs-gitlab-runner ~]# cat /path/to/config/config.toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "docker-runner"
  url = "http://gitlab.samchu.com/"
  token = "ckfxxxxxxm6"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "alpine:3"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
```
這是原始的  

接下來為了在 Docker 中可以包 Docker, 需要做點手腳從宿主主機的 Docker 拉進來使用, 所以我們改一下設定檔跟他說 Docker 在哪
```
[root@cs-gitlab-runner ~]# sed -i -e '/volumes/d' \
>   -e '/^    shm_size/a\    volumes = ["/cache", "/var/run/docker.sock:/run/docker.sock"]' \
>   -e '/^    shm_size/a\    network_mode = "host"' /path/to/config/config.toml
```

執行完後的檔案變這樣
```
[root@cs-gitlab-runner ~]# cat /path/to/config/config.toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "docker-runner"
  url = "http://gitlab.samchu.com/"
  token = "ckxxxxCm6"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "alpine:3"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    shm_size = 0
    volumes = ["/cache", "/var/run/docker.sock:/run/docker.sock"]
    network_mode = "host"
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
```

確認有成功修改到後, 就可以執行我們的 Runner  
```
[root@cs-gitlab-runner ~]# docker run -d --name gitlab-runner --restart always \
>    -v /path/to/config:/etc/gitlab-runner \
>    -v /var/run/docker.sock:/var/run/docker.sock \
>    gitlab/gitlab-runner:latest
584e39680fa0e35535ccb6688584639bed308ea0e41e9d17e183c288da8e6d6c
```

順利的話, 你就可以看到你的 Runner 是綠燈狀態囉  

接下在你要打包的那個專案 Enable shared Runners 就可以使用 Gitlab 共用的 Runner 囉

下一篇說明 .gitlab-ci.yml 如何寫腳本給 Runner 執行

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。