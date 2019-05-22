---
title: Ansible 安裝及入門
tags:
  - Ansible
categories:
  - DevOps
---


<!--more-->

安裝 Ansible
``` bash
sudo yum install epel-release -y
sudo yum update -y

# install python-pip from epel
sudo yum install python-pip -y
# Verify using:
sudo pip -V
# Upgrade pip
sudo pip install --upgrade pip

sudo yum install ansible -y
sudo ansible --version

sudo pip install ansible-lint
sudo ansible-lint --version
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。