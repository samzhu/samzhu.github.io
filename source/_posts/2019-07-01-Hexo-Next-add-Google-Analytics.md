---
title: Hexo Next add Google Analytics
tags:
  - Hexo
categories:
  - 日常
date: 2019-07-01 15:40:51
---


如何在 Hexo 的 Next 模板增加 Google Analytics?

<!--more-->

因為需要修改配置檔 /blog/themes/next/_config.yml  
所以就把 CI 中的 git clone 註解掉了  
```
#      - run: git clone https://github.com/theme-next/hexo-theme-next themes/next
```
改成自己下載放到 /blog/themes/next  

然後 修改 /blog/themes/next/_config.yml  
``` yml
# Google Analytics
google_analytics:
  tracking_id: UA-123456789-1
  localhost_ignored: false
```

大致上就完成了

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。