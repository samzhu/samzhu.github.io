---
title: 註冊 GitLab Runner use Shell
tags:
  - GitLab
categories:
  - DevOps
date: 2019-05-21 12:18:26
---


原本是在 Runner 內使用 ssh 直接去做佈署, 但是這樣很沒有效率 失誤率也很高, 所以之後會打算使用 Ansible 來做佈署動作, 不過不管哪一種都是需要使用將GitLab Runner 安裝在宿主主機上, 而不是透過 (GitLab Runner in Docker)[https://blog.samchu.dev/2019/05/02/%E8%A8%BB%E5%86%8A-GitLab-Runner-use-Docker/] 的這種方式, 因為不管要存取 SSH 金鑰還是鑰呼叫 ansible-playbook 都是直接安裝在宿主主機比較方便啊.

這邊就簡單介紹如何配置跟註冊, 並使用簡單的 shell 來操作遠端機器

<!--more-->

## Install GitLab Runner using the official GitLab repositories
教學網址 (Install GitLab Runner using the official GitLab repositories)[https://docs.gitlab.com/runner/install/linux-repository.html]  

我是使用 CentOS 就按照步驟做吧  

Add GitLab’s official repository
``` bash
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
```

Install the latest version of GitLab Runner
``` bash
sudo yum install -y gitlab-runner
```

好的, 再來是註冊 runner
```
sudo gitlab-runner register \
  --non-interactive \
  --url "https://dev-gitlab.samchu.com/" \
  --registration-token "12345678901qaz2wsx" \
  --executor "shell" \
  --description "dev-docker-runner" \
  --run-untagged \
  --locked="false"
```
註冊成功後就會寫入到 /etc/gitlab-runner/config.toml  

如果需要移除所有 Runner
```
sudo gitlab-runner unregister --all-runners
```

或是暴力一點直接殺檔案, 之後再到網頁上自己移除
```
sudo rm /etc/gitlab-runner/config.toml
```

## 問題排除

1.Runner Clone 位置錯誤

如果你發現 runner 活著, 但它 git clone 的位置老是錯誤, 有可能是一開始的 GitLab 就認錯了, 所以 Runner 也會記錯, 解決方法是 提供給它 clone 的正確位置
```
sudo sed -i \
  -e '/^  executor/a\  clone_url = "https://dev-gitlab.samchu.com"' /etc/gitlab-runner/config.toml

sudo gitlab-runner restart
```

2.Runner 執行時預設的 User

預設會建一個 gitlab-runner 用戶來執行, 可以移除後再安裝並指定用戶為 centos, 但是某個時間點又會跳回 gitlab-runner 導致腳本錯誤, 所以這做法就不採用了,如果有需要可參考下面指令
```
sudo gitlab-runner uninstall

sudo gitlab-runner install --working-directory /home/centos --user centos

sudo systemctl daemon-reload
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。