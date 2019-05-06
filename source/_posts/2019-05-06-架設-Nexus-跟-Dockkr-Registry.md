---
title: 架設 Nexus 跟 Dockkr Registry
tags:
  - DockerCompose
  - Nexus
categories:
  - DevOps
date: 2019-05-06 11:51:28
---


既然程式已經編譯打包好了, 那就需要有個地方交付儲存,  
以前是交付 Jar, 但現在都是用 Docker 了, 所以我們交付的完成品就是 Docker Image,  
那我們交付的儲存庫一種是用 docker registry 這個官方有出, 一些大公司有做開源也可以直接拿來用,  
不過我們本來就有用 Nexus 了, 那直接用 Nexus 來管 docker registry 當然是最方便的不二人選啊.

<!--more-->

安裝方式分兩種, 一種手動 一種用 Ansible 安裝  (後面講 Ansible 入門 再說明 Ansible)
先介紹手動的, 後面有介紹 Ansible 版, 但其實安裝步驟是一樣的, 最後再說明服務啟動後需要上介面做什麼設定.

## 手動安裝

### 手動初始化
請參考 [如何初始化 CentOS](https://blog.samchu.dev/2019/04/30/%E5%A6%82%E4%BD%95%E5%88%9D%E5%A7%8B%E5%8C%96CentOS/)   

### 建立&啟動
使用 docker-compose 建立
```
sudo mkdir -p /usr/local/lab
sudo chown -R centos /usr/local/lab
mkdir -p /usr/local/lab/nexus/data && chmod 777 data
cd /usr/local/lab/nexus

cat << 'EOF' > docker-compose.yml
version: '3.7'
services:
  nexus:
    image: 'sonatype/nexus3'
    container_name: 'nexus'
    restart: always
    ports:
      - '80:8081'
      - '15000:15000'
    volumes:
      - '/etc/timezone:/etc/timezone'
      - '/etc/localtime:/etc/localtime'
      - '/usr/local/lab/nexus/data:/nexus-data'
EOF

sudo docker-compose up -d
```

## Ansible 安裝
首先要透過 Ansible 來安裝避必須透過 git submodule 來關聯到 ansible 專案, 因為那邊有機器初始化腳本可以一併處理  

### 執行 ansible-playbook
透過以下指令初始化  
```
ansible-playbook -i inventory ansible/playbooks/initial.yml --limit 'group_nexus' -vv
```
初始化指令執行完後會進行重開機  
重複執行沒有關係, 初始化完成後會寫一個用戶變數, 下次執行發現已執行過了就不會再執行重開機指令  

接著進行安裝
```
ansible-playbook -i inventory ansible/roles/nexus/tasks/main.yml --limit 'group_nexus' -vv
```

結束

### Git submodule 處理

增加子模組做關聯
```
git submodule add ../../customer-service/ansible.git
git status
git commit -a -m "first commit with submodule ansible"
git push
```

子模組有更新
```
git submodule update --remote
git status
git commit -a -m "update with submodule ansible"
git push
```

## 系統設定

### Create Blob Stores
![Imgur](https://i.imgur.com/M44UJPh.png)
![Imgur](https://i.imgur.com/acwIGkG.png)
![Imgur](https://i.imgur.com/hqSgbHt.png)
![Imgur](https://i.imgur.com/xxcYPla.png)

### Create Repositories
![Imgur](https://i.imgur.com/hnvN3qP.png)
![Imgur](https://i.imgur.com/ZOMzTyt.png)
![Imgur](https://i.imgur.com/QU8y0N7.png)
![Imgur](https://i.imgur.com/dcc8pCu.png)

### Create Role for image push
![Imgur](https://i.imgur.com/hudqqZz.png)
![Imgur](https://i.imgur.com/E2oQlCn.png)
![Imgur](https://i.imgur.com/ghzJ279.png)

### Create User for image push
![Imgur](https://i.imgur.com/w10ouWC.png)
![Imgur](https://i.imgur.com/BoOoaDV.png)
![Imgur](https://i.imgur.com/ID44BP0.png)

### Create Role for image pull (read only)
![Imgur](https://i.imgur.com/kP9QRk8.png)

### Create User for image pull (read only)
![Imgur](https://i.imgur.com/L2DQW1W.png)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/tw/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/3.0/tw/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記</span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/tw/">創用CC 姓名標示-非商業性-相同方式分享 3.0 台灣 授權條款</a>釋出。