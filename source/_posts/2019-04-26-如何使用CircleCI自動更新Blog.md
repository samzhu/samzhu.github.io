---
title: 如何使用CircleCI自動更新Blog
date: 2019-04-26 17:57:32
categories:
  - 日常
tags:
  - Hexo
  - GitHub Pages
  - CircleCI
---

科技來自於惰性, 當然更新也需要自動化啦  

所以利用免費服務來自動幫我 [CircleCI](https://circleci.com), 來更新 GitHub Page 啦.  

<!--more-->

## 建立 CI 檔案
CI 的資料夾
```
mkdir .circleci
```

CI 的腳本檔
```
cat << 'EOF' > .circleci/config.yml
# Javascript Node CircleCI 2.0 configuration file
#
# Check https://circleci.com/docs/2.0/language-javascript/ for more details
#
version: 2
jobs:
  build:
    filters:
      branches:
        ignore:
          - master

    docker:
      - image: circleci/node:10.15.3
    
    working_directory: ~/repo

    steps:

      - add_ssh_keys:
          fingerprints:
            - "b9:94:f2:ff:40:5d:d5:6d:42:7d:bb:84:77:34:ef:5f"

      - checkout

      # Download and cache dependencies
      - restore_cache:
          keys:
            - v1-dependencies-{{ checksum "package.json" }}
            # fallback to using the latest cache if no exact match is found
            - v1-dependencies-

      - run: npm install
      - run: sudo npm install hexo-cli --save -g
      - run: sudo npm install hexo-deployer-git --save -g

      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package.json" }}

      - run: ls ./source/_posts

      - run:
          name: set git info
          command: |
                git config --global user.email "spike19820318@gmail.com"
                git config --global user.name "sam.chu"

      - run:
          name: hexo clean
          command: |
                hexo clean

      - run:
          name: hexo generate
          command: |
                hexo generate

      - run:
          name: hexo deploy
          command: |
                hexo deploy
EOF
```

腳本的內容就不詳細說明了, 不過比較特別的是 add_ssh_keys 這個動作  

因為 CircleCI 跟 Github 只有拿唯讀的 權限 Token, 所以當你要做 hexo deploy 的時候會發現是失敗的.

這邊你要回到 CircleCI 的網站, 透過 CircleCI 再去跟 Github 拿一把完整的 user token, 這樣你才能發佈回去 Github.

進到設定, 選 Checkout SSH keys
![Imgur](https://i.imgur.com/iyk1ixK.png)

現在可以看到所謂 Deploy key 而這把 key 是唯讀的
![Imgur](https://i.imgur.com/So0FHtP.png)

我們點選下面的 Authorize With GitHub 按鈕 進入 GitHub 做授權
![Imgur](https://i.imgur.com/jeCiqe0.png)

完成之後就可以把原本的 Deploy key 刪掉了
![Imgur](https://i.imgur.com/hYMA78s.png)

並將將特徵碼填入到 .circleci/config.yml 的 add_ssh_keys.fingerprints

這樣一來, Hexo 就可以順利發佈啦

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/tw/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/3.0/tw/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記</span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/tw/">創用CC 姓名標示-非商業性-相同方式分享 3.0 台灣 授權條款</a>釋出。