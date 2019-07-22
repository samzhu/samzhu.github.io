---
title: Ansible install and introduction
tags:
  - Ansible
categories:
  - DevOps
date: 2019-07-05 11:58:04
---


現在要來把之前機器初始化的手動腳本, 改成自動化

如果你是 Mac 又沒裝 XCode 可參考做法 [Install Ansible on Mac](https://blog.samchu.dev/2019/07/02/Install-Ansible-on-Mac/)  

如果是 CentOS 就順著往下做就可以

<!--more-->

## 安裝 Ansible  

### 先更新
``` bash
sudo yum install epel-release -y
sudo yum update -y
```

### Install python-pip from epel
``` bash
sudo yum install python-pip -y
```
### Verify using
``` bash
sudo pip -V
```
### Upgrade pip
``` bash
sudo pip install --upgrade pip
```

### Install Ansible
``` bash
sudo yum install ansible -y
sudo ansible --version
```

### Install Verify Tool
``` bash
sudo pip install ansible-lint
sudo ansible-lint --version
```

到這一步我們控制的主機基本上是裝好了  
接下來我們來初始化 Ansible Controller 資料夾

## Ansible config file
配置文件 按優先級排序如下：  
- ANSIBLE_CONFIG (環境變量，如果不為空則最優先)  
- ansible.cfg (當前路徑下的配置)  
- ~/.ansible.cfg (用戶主目錄下)  
- /etc/ansible/ansible.cfg （全局配置文件）  

CentOS 裝好預設是會有建立目錄
``` bash
$ ls -lh /etc/ansible
total 24K
-rw-r--r--. 1 root root  20K Feb 21 18:04 ansible.cfg
-rw-r--r--. 1 root root 1016 Feb 21 18:04 hosts
drwxr-xr-x. 2 root root    6 Feb 21 18:04 roles
```
ansible.cfg  => Setting ansible執行roles路徑及其他設定功能
hosts        => Setting ssh 遠端vm
roles        => defule 角色

如果沒有的話也可以下載範例來修改
``` bash
curl -sSL https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg -o ~/.ansible.cfg
```

## Inventory
受 Ansible 管理的主機列表  
格式如下  
```
[group_eureka]
eureka-01 ansible_host=192.168.0.11 ansible_user=root
eureka-02 ansible_host=192.168.0.12 ansible_user=root

[group_config]
config_01 ansible_host=192.168.0.31 ansible_user=root
config_02 ansible_host=192.168.0.32 ansible_user=root
```
方括弧內的就是所謂群組, 你可以一次指定一個群組, 或是單一一台主機名  
例如  

指定 eureka-01 一台
```
ansible-playbook playbooks/PLAYBOOK_NAME.yml --limit "eureka-01"
```

指定一個群組 group_config 主機 config_01 & config_02 都會執行
```
ansible-playbook playbooks/PLAYBOOK_NAME.yml --limit 'group_config'
```

做個練習讓 Ansible 去 ping 看看所有主機

``` bash
cat << 'EOF' > inventory
[group_eureka]
eureka-01 ansible_host=192.168.0.11 ansible_user=root
eureka-02 ansible_host=192.168.0.12 ansible_user=root

[group_config]
config_01 ansible_host=192.168.0.31 ansible_user=root
config_02 ansible_host=192.168.0.32 ansible_user=root
EOF
```

假設 eureka-01 & config_01 都是正常可以 ping 得到 結果應該類似如下就成功了
``` bash
ansible -i ./inventory group_config -m ping 
config_02 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: connect to host 192.168.0.32 port 22: Operation timed out",
    "unreachable": true
}
config_01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

其他說明參考官網 [Ansible Limiting Playbook/Task Runs](https://ansible-tips-and-tricks.readthedocs.io/en/latest/ansible/commands/#limiting-playbooktask-runs)

## Ansible initialization folder
要透過 ansible 操作當然免不了要分資料夾來做管理不同環境
你可以參考官方 [Ansible Best Practices Directory Layout](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout),  
或是 [凍仁翔](https://note.drx.tw/) 大大的實戰分享 [現代 IT 人一定要知道的 Ansible 自動化組態技巧 19. 如何維護大型的 Ansible Playbooks？](https://ithelp.ithome.com.tw/articles/10186729)
[](https://ansible-tran.readthedocs.io/en/latest/index.html)

我簡化一下只列出這次練習會用得到的部分, 完整的到官網看吧
```
production                # 正式環境主機管理檔
staging                   # 測試環境主機管理檔

group_vars/               # 群組變數
   all.yml                # 各環境共用的群組變數
   production.yml         # 正式環境的群組變數
   staging.yml            # 測試環境的群組變數
host_vars/                # 主機變數
   hostname1.yml          # here we assign variables to particular systems
   hostname2.yml

library/                  # if any custom modules, put them here (optional)
module_utils/             # if any custom module_utils to support modules, put them here (optional)
filter_plugins/           # if any custom filter plugins, put them here (optional)

site.yml                  # master playbook
webservers.yml            # playbook for webserver tier
dbservers.yml             # playbook for dbserver tier

roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case

    webtier/              # same kind of structure as "common" was above, done for the webtier role
    monitoring/           # ""
    fooapp/               # ""
```

在範例資料夾來建立一些目錄讓他跟官方建議的目錄比較接近
```
cat << 'EOF' > README.md
Ansible 腳本範例
EOF

# 開發環境
cat << 'EOF' > dev
[group_eureka]
eureka-01 ansible_host=192.168.1.11 ansible_user=root
eureka-02 ansible_host=192.168.1.12 ansible_user=root

[group_config]
config_01 ansible_host=192.168.1.31 ansible_user=root
config_02 ansible_host=192.168.1.32 ansible_user=root
EOF

# 開發預發佈環境
cat << 'EOF' > dev-release
[group_eureka]
eureka-01 ansible_host=192.168.2.11 ansible_user=root
eureka-02 ansible_host=192.168.2.12 ansible_user=root

[group_config]
config_01 ansible_host=192.168.2.31 ansible_user=root
config_02 ansible_host=192.168.2.32 ansible_user=root
EOF

mkdir group_vars

mkdir roles

mkdir files
```

## Playbook

### 第一個 playbook
建立初始化 playbook, 但是剛開始練首先用簡單一點的 印出主機名稱
```
cat << 'EOF' > initial.yml
- name: Ansible Check
  hosts: "{{ myhosts }}"
  serial: 1
  tasks:
    - name: echo whoami
      command: whoami
      register: whoami_result

    - name: print whoami result
      debug:
        msg: "{{ whoami_result.stdout }}"

    - name: echo hostname
      command: hostname
      register: hostname_result

    - name: print hostname result
      debug:
        msg: "{{ hostname_result.stdout }}"
EOF
```

目前結構
```
$ tree
.
├── README.md
├── dev
├── dev-release
├── group_vars
├── initial.yml
└── roles

2 directories, 4 files
```

如何執行, 重要的參數使用系統變數來讀入執行
```
export myhosts=all
export ansible_password=1qaz2wsx
ansible-playbook -i ./dev ./initial.yml -e "myhosts=${myhosts}" -vv --extra-vars "ansible_password=${ANSIBLE_PASSWORD}"
```

### 免詢問登入
如果你還沒有修改免確認登入會變成卡在這邊
``` bash
$ ansible-playbook -i ./dev ./initial.yml -e "myhosts=${myhosts}" -vv --extra-vars "ansible_password=${ANSIBLE_PASSWORD}"
ansible-playbook 2.8.1
  config file = None
  configured module search path = ['/Users/shanglichu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/Cellar/ansible/2.8.1_1/libexec/lib/python3.7/site-packages/ansible
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.7.3 (default, Jun 19 2019, 07:38:49) [Clang 10.0.1 (clang-1001.0.46.4)]
No config file found; using defaults

PLAYBOOK: initial.yml ****************************************************************************************************************************************************
1 plays in ./initial.yml

PLAY [Ansible Check] *****************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************
task path: /ansible-demo/initial.yml:1
The authenticity of host '10.0.29.95 (10.0.29.95)' can't be established.
ECDSA key fingerprint is SHA256:rfeIQ8KcFwSk24fnjYelPAb1wXFuL4Byok0hPXMzylQ.
Are you sure you want to continue connecting (yes/no)?
```

依照官方文件[Host Key Checking](https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html#host-key-checking)  
你可以在設定檔 /etc/ansible/ansible.cfg or ~/.ansible.cfg 配置
```
[defaults]
host_key_checking = False
```

或是配在環境變數
```
export ANSIBLE_HOST_KEY_CHECKING=False
```

### 缺 sshpass 工具
接下來你可能會遇到這個錯
``` bash
$ ansible-playbook -i ./dev ./initial.yml -e "myhosts=${myhosts}" -e ansible_password=1qaz2wsx

PLAY [Ansible Check] *****************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************
fatal: [eureka-01]: FAILED! => {"msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}

PLAY RECAP ***************************************************************************************************************************************************************
eureka-01                  : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```

安裝方式
``` bash
# CentOS
yum -y install sshpass

# Mac
brew install https://raw.githubusercontent.com/kadwanev/bigboybrew/master/Library/Formula/sshpass.rb

# 網路找到另一個地方 程式碼一樣
brew install http://git.io/sshpass.rb
```

Mac 執行結果
``` bash
$ brew install https://raw.githubusercontent.com/kadwanev/bigboybrew/master/Library/Formula/sshpass.rb
Updating Homebrew...
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/core).
==> Updated Formulae
angular-cli        bazel              devspace           exploitdb          grpc               libuv              neo4j              skaffold           wireguard-tools
argon2             blink1             dub                gatsby-cli         kubeless           mujs               paket              terragrunt

######################################################################## 100.0%
==> Downloading http://sourceforge.net/projects/sshpass/files/sshpass/1.06/sshpass-1.06.tar.gz
==> Downloading from https://nchc.dl.sourceforge.net/project/sshpass/sshpass/1.06/sshpass-1.06.tar.gz
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/sshpass/1.06
==> make install
🍺  /usr/local/Cellar/sshpass/1.06: 9 files, 41.6KB, built in 24 seconds
```

### 執行結果
再執行一次
``` bash
$ export myhosts=all
$ export ansible_password=1qaz2wsx
$ export ANSIBLE_HOST_KEY_CHECKING=False
$ ansible-playbook -i ./dev ./initial.yml -e "myhosts=${myhosts}" -e "ansible_password=${ansible_password}"

PLAY [Ansible Check] *****************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************
ok: [eureka-01]

TASK [echo whoami] *******************************************************************************************************************************************************
changed: [eureka-01]

TASK [print whoami result] ***********************************************************************************************************************************************
ok: [eureka-01] => {
    "msg": "root"
}

TASK [echo hostname] *****************************************************************************************************************************************************
changed: [eureka-01]

TASK [print hostname result] *********************************************************************************************************************************************
ok: [eureka-01] => {
    "msg": "localhost.localdomain"
}

PLAY RECAP ***************************************************************************************************************************************************************
eureka-01                  : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
這樣我們就可以順利操作遠端的機器, 並取得登入帳號跟 hostname 了

## Role
當你要配置的功能種類越來越多的時候, Role 會是比較好管理的選擇  

初始化特定角色資料夾
``` bash
$ ansible-galaxy init roles/springboot
- roles/springboot was created successfully
```
這樣我們就有一個 springboot 角色資料夾, 有關 springboot 部署要用的檔案或配置就可以放這了

完整結構
```
$ tree
.
├── README.md
├── dev
├── dev-release
├── group_vars
├── initial.yml
└── roles
    └── springboot
        ├── README.md
        ├── defaults
        │   └── main.yml
        ├── files
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        ├── templates
        ├── tests
        │   ├── inventory
        │   └── test.yml
        └── vars
            └── main.yml

11 directories, 12 files
```

## 如何練習
基本的介紹完了  
要開始正式寫腳本, 會需要常常重置 VM, 蠻推薦使用  
[VirtualBox](https://www.virtualbox.org/)  加上 [Vagrant](https://www.vagrantup.com/)  
來當作一個組合技, 非常之好用.

## References
[現代 IT 人一定要知道的 Ansible 自動化組態技巧 19. 如何維護大型的 Ansible Playbooks？](https://ithelp.ithome.com.tw/articles/10186729)
[Ansible中文权威指南](https://ansible-tran.readthedocs.io/en/latest/index.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。