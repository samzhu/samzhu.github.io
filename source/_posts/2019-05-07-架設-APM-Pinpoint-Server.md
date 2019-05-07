---
title: 架設 APM Pinpoint Server
date: 2019-05-07 18:03:33
tags:
---


Pinpoint 是 Line 背後公司開源的一種 Application Performance Management (APM)  
[Pinpoint 官網](https://naver.github.io/pinpoint/) 在這裡  
這一篇教你如何架設 Pinpoint Server 來收集應用的資訊  

<!--more-->

Pinpoint Server 安裝

預先安裝
Docker
Docker Compose

如果沒有 unzip 就加裝一下
```
sudo yum install -y unzip
sudo apt-get install unzip
```

sudo yum install -y wget


安裝 Pinpoint
```
wget --no-check-certificate https://github.com/naver/pinpoint-docker/archive/master.zip -O ./pinpoint1.8.1.zip
unzip pinpoint1.8.1.zip -d .

cd pinpoint-docker-master
```

移除 pinpoint-quickstart, pinpoint-agent 部分
```
sed -i '148,191d' docker-compose.yml 
```

Web 介面改用 80 port
```
sed -i 's/WEB_PAGE_PORT=8079/WEB_PAGE_PORT=80/g' .env
```

啟動
```
docker-compose up -d
```

測試一下
```
curl localhost
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/tw/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/3.0/tw/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記</span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/tw/">創用CC 姓名標示-非商業性-相同方式分享 3.0 台灣 授權條款</a>釋出。