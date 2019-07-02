---
title: Install Ansible on Mac
tags:
  - ansible
categories:
  - DevOps
date: 2019-07-02 10:08:17
---


如何在 Mac 上安裝 Ansible?

<!--more-->

## 使用 Homebrew

### Install Homebrew

首先先裝 Mac 是好用的軟體管理工具 [Homebrew](https://brew.sh/)
``` bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Install Ansible

``` bash
brew install ansible
```

### Test
``` bash
$ python --version
Python 2.7.10

$ python3 --version
Python 3.7.3

$ ansible --version
ansible 2.8.1
  config file = None
  configured module search path = ['/Users/shanglichu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/Cellar/ansible/2.8.1_1/libexec/lib/python3.7/site-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.7.3 (default, Jun 19 2019, 07:38:49) [Clang 10.0.1 (clang-1001.0.46.4)]
```

## 使用 pip
這方案必須先裝 Xcode, 所以我就沒試了


## References
[Install Ansible on Mac OSX](https://hvops.com/articles/ansible-mac-osx/)



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。