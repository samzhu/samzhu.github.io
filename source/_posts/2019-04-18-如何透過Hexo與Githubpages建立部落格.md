---
title: 如何透過 Hexo 與 Githubpages 建立部落格
date: 2019-04-18 13:31:45
categories:
  - 日常
tags:
  - Hexo
  - GitHub Pages
---
這篇教你如何用 Hexo 跟 GitHub Pages 建立部落格或履歷頁面XD

原本部落格[http://samchu.logdown.com](http://samchu.logdown.com)就只是一個筆記的地方, 沒有想管他好不好用, 但是越來越覺得它快倒站啦XD  
只好趁這機會一起整理整理搬家, 第一篇當然就是如何建立部落格啦XD

## 首先必備
1. Github 帳號
1. Google Domain or 其他家網域

<!--more-->

##  安裝 NVM
[安裝參考連結](https://github.com/creationix/nvm#install--update-script)

執行
``` bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
```
關閉終端再開啟  
  
驗證
``` bash
nvm --version
```

查看所有可以安裝的LTS版本（長期支持版）
``` bash
nvm ls-remote --lts
```
安裝指定版本
``` bash
nvm install v10.15.3
```
使用指定版本
``` bash
nvm use v10.15.3
```

## 安裝 Hexo
[安裝參考連結](https://hexo.io/docs/)  
[中文站](https://hexo.io/zh-tw/docs/)  

使用 npm 安裝
``` bash
npm install -g hexo-cli
```

建立一個新的網站。如果沒有設定 folder 的話，Hexo 會在目前的資料夾建立網站。
``` bash
export targetDir=<資料夾名稱>
hexo init ${targetDir}
cd ${targetDir}
npm install
```
我的目錄是設定在 /github/blog

使用預設面板發生錯誤
```
The tag fancybox on line 77 in themes/landscape/README.md is not a recognized Liquid tag
```
因為語法問題, 移除 themes/landscape/README.md 或註解那一行

下面列出幾個比較常用指令
``` bash
# 本機預覽
hexo server
# 清除快取檔案 (db.json) 和已產生的靜態檔案 (public)
hexo clean
# 產生靜態檔案
hexo generate
```

到上面為止, 基本上 Hexo 基本使用 OK 了, 就可以開始配置串接部分

## 串接 Github Pages
如果你是要申請個人的 GithubPage , 請注意頁面的分支不會是使用 gh-pages 喔.  
詳細請參考說明 [Configuring a publishing source for GitHub Pages](https://help.github.com/en/articles/configuring-a-publishing-source-for-github-pages)  
個人的 GithubPage 來源是被限定在 master 分支, 也就是說 你在 master 上放一個 index.html 馬上就可以呈現了.  

所以 Hexo 也要反過來調整一下.

### 建立個人頁 Repository
像我的 Github 名稱是 samzhu, 那我就必須建立一個 samzhu.github.io 的儲存庫, 建立好之後在 Github setting 像這樣

![2019-04-17](https://i.imgur.com/vvD8Ule.png)

立馬就可以發佈頁面在 https://samzhu.github.io/
但是注意一下, 個人頁面的 source 是只能從 master 來的喔, 這點跟一般專案頁面不太一樣

接下來初始化我們機器上的 Git 吧

### 配置 git
還記得前面說過, 個人頁是發佈在 master 吧, 所以我在本地另外建立分支叫 hexo , 原始檔就都存在這分支, 而發佈就讓 Hexo 發到 master 就完成了

先回到 Hexo Blog 的根目錄 /github/blog

初始化 Git
``` bash
git init

git checkout -b hexo
# 等效下面兩個指令
# git branch hexo
# git checkout hexo

git add .
git commit -m "git init"
```

### push an existing repository
``` bash
# 儲存庫路徑
git remote add origin https://github.com/samzhu/samzhu.github.io.git
# branch:master
git push -u origin master
```

### 修改 Ｈexo 配置檔

/github/blog/_config.yml
``` yml
url: https://samzhu.github.io

deploy:
  type: git
  repo: https://github.com/samzhu/samzhu.github.io.git
  branch: master
```

### push(deploy)

install hexo自動部署工具
``` bash
npm install hexo-deployer-git --save
```

push public to branch: _config.yml deploy.branch
``` bash
hexo d
# or hexo deploy
```

## Using a custom domain with GitHub Pages

[Using a custom domain with GitHub Pages](https://help.github.com/en/articles/using-a-custom-domain-with-github-pages)  

到這步驟為止呢, 已經完成了在 GitHub Pages 上的個人頁面, 其實要當當履歷簡介或是筆記都很夠用方便啦, 但身為工程師, 打滾這麼多年, 如果有一個 個人網域 ~~工程宅的浪漫~~ 多酷啊  

我自己是買 google domain 服務, 趁 dev 網域剛開就買了
![Imgur](https://i.imgur.com/yB0ytLm.png)

其實就增加 CNAME 的 blog 只到 github 給的子網域

然後回到 Github setting 頁面
把你預計想要使用的 domain 輸入在 Custom domain 儲存
![Imgur](https://i.imgur.com/5NYMzHF.png)

好了之後就會看到 Your site is published at 已經變成你的域名了
![Imgur](https://i.imgur.com/jiPxmYT.png)

這時侯再打勾 Enforce HTTPS 就變成 https 了
![Imgur](https://i.imgur.com/FftoYSf.png)

是不是超簡單的呢?

上面這些事情啊, Github 幫我們都簡化了  

Custom domain 設定時 github 就會幫我們新增一個 CNAME 檔到 master 分支上, 但如果我們用 hexo deploy 後就會全部被洗掉, 當然 CNAME 也會沒了, 網域又失效了  

所以我可以下載 github 幫我做好的 CNAME 放到 /github/blog/source  
這樣 hexo generate 的時候就會幫你放到 public 的目錄了

Enforce HTTPS 則是 Github 跟 Let's Encrypt 合作的 從申請到完成都是自動化, 所以你也省去了自己管憑證的麻煩


## 新增寫作

```
hexo new [layout] <title>

hexo new 如何透過Hexo與Githubpages建立部落格
```

就會在 source/_posts 下建立新的 MD
/github/blog/source/_posts/如何透過Hexo與Githubpages建立部落格.md

## 參考資料

[使用 Github Pages + Hexo 搭建个人博客](https://gelomen.github.io/Hexo/%E4%BD%BF%E7%94%A8Github-Pages-Hexo-%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2.html)

[從零開始: 用github pages 上傳靜態網站](https://medium.com/進擊的-git-git-git/從零開始-用github-pages-上傳靜態網站-fa2ae83e6276)

[使用 Google Domains 註冊個人網域](https://blog.jaycetyle.com/2018/01/google-domain/)

[Setting up an apex domain](https://help.github.com/en/articles/setting-up-an-apex-domain#configuring-a-records-with-your-dns-provider)

[Hexo + Github Pages：教你設定部落格的專屬網址，含網域購買教學](https://www.larrynote.com/website-service/50343/)

[How to setup google domain for github pages](https://trentyang.com/how-to-setup-google-domain-for-github-pages/)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。