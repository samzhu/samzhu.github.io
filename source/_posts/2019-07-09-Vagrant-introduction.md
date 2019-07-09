---
title: Vagrant introduction
tags:
  - Vagrant
  - VirtualBox
categories:
  - DevOps
date: 2019-07-09 09:28:53
---


要做一些機機器的管理部署, 會很常需要砍掉再重建, 如果你每次都要從安裝開始, 真的是非常花費時間.  
你要說用 快照機制 也不是不行, 但大家 Lab 的基礎環境就又會不太一樣, 你就會常常遇到 "it work on my machine" !!  

我們利用 Vagrant 來快速建構跟管理標準化且一致的 Lab 環境吧.

<!--more-->

其實說穿了 Vagrant 就是一個管理 VM 的一個工具, 當你需要什麼 OS 其實大部分都有人做好了基礎 BOX 也有官方版的 CentOS or Ubuntu, 只要下載下來就可以用, 這點非常方便.

## VirtualBox
請到 [Oracle VM VirtualBox](https://www.virtualbox.org/) 下載安裝  
目前的版本應為 6.0.8

## Vagrant
請到 [Vagrant by HashiCorp](https://www.vagrantup.com/) 下載安裝  
目前的版本應為 2.2.5

## 使用說明
上面 VirtualBox 跟 Vagrant 應該抓主要穩定版本都沒啥問題, 除非你不小心抓到 beta 版什麼的, 到時候就刪除再抓別的的版本試試看.  

那使用方式就是找個資料夾 裡面放個 Vagrantfile 檔案就可以了  

最簡單的一個 VM 如下
Vagrantfile
``` Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "myvm" do |myvm|
    myvm.vm.box = "centos/7"
    myvm.vm.hostname = 'vm001'
  end
end
```

接著在檔案鎖在資料夾下指令
``` bash
$ vagrant up
Bringing machine 'myvm' up with 'virtualbox' provider...
==> myvm: Importing base box 'centos/7'...
==> myvm: Matching MAC address for NAT networking...
==> myvm: Checking if box 'centos/7' version '1905.1' is up to date...
==> myvm: Setting the name of the VM: demo-01_myvm_1562554459011_52775
==> myvm: Clearing any previously set network interfaces...
==> myvm: Preparing network interfaces based on configuration...
    myvm: Adapter 1: nat
==> myvm: Forwarding ports...
    myvm: 22 (guest) => 2222 (host) (adapter 1)
==> myvm: Booting VM...
==> myvm: Waiting for machine to boot. This may take a few minutes...
    myvm: SSH address: 127.0.0.1:2222
    myvm: SSH username: vagrant
    myvm: SSH auth method: private key
    myvm:
    myvm: Vagrant insecure key detected. Vagrant will automatically replace
    myvm: this with a newly generated keypair for better security.
    myvm:
    myvm: Inserting generated public key within guest...
    myvm: Removing insecure key from the guest if it's present...
    myvm: Key inserted! Disconnecting and reconnecting using new SSH key...
==> myvm: Machine booted and ready!
==> myvm: Checking for guest additions in VM...
    myvm: No guest additions were detected on the base box for this VM! Guest
    myvm: additions are required for forwarded ports, shared folders, host only
    myvm: networking, and more. If SSH fails on this machine, please install
    myvm: the guest additions and repackage the box to continue.
    myvm:
    myvm: This is not an error message; everything may continue to work properly,
    myvm: in which case you may ignore this message.
==> myvm: Setting hostname...
==> myvm: Rsyncing folder: /vagrant/demo-01/ => /vagrant
```
看到這樣的畫面表示 VM 的 BOX 已經下載並啟動完成, 這樣你就有一個標準的 CentOS, 接下來我們 ssh 進去 VM.  

``` bash
$ vagrant ssh
[vagrant@vm001 ~]$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
[vagrant@vm001 ~]$ hostname
vm001
```
是不是一分鐘搞定, 而且大家的 VM 應該都是一致的, 不會有安裝上的差異.

## Vagrant 指令
這些指令可以自己玩玩看  

vagrant init / 初始化
vagrant up / 啟動
vagrant status / 查看虛擬機的狀態(關機、開機)
vagrant ssh-config / 查看基本登入資訊
vagrant ssh / 進入虛擬機
vagrant halt / 關機
vagrant destroy / 刪除
vagrant destroy -f / 直接移除

## 同步資料夾
通常 Vagrant 會將執行所在的資料夾掛載到 VM 中  
ex:  
``` bash
==> controller: Rsyncing folder: /vagrant/demo-05/ => /vagrant
```
/vagrant/demo-05/ 就是我指令執行的所在目錄, /vagrant 就是對應到 VM 內的路徑  

如果宿主主機上有檔案需要放在 VM 中, 可以寫成這樣的 Vagrantfile  
``` Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "myvm" do |myvm|
    myvm.vm.box = "centos/7"
    myvm.vm.synced_folder "/github", "/tmp/github"
  end
end
```

但執行下去馬上 GG  
``` bash
    myvm: /tmp/github => /github
Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000 srv_github /srv/github

The error output from the command was:

mount: unknown filesystem type 'vboxsf'
```

這時候只要加裝 plugin 就可以解決了    
plugin 安裝部分 mac 跟 win 都是一樣.
``` bash
$ vagrant plugin install vagrant-vbguest
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Fetching: micromachine-2.0.0.gem (100%)
Fetching: vagrant-vbguest-0.18.0.gem (100%)
Installed the plugin 'vagrant-vbguest (0.18.0)'!
```

再次執行
``` bash
$ vagrant up
Bringing machine 'myvm' up with 'virtualbox' provider...
==> myvm: Importing base box 'centos/7'...
==> myvm: Matching MAC address for NAT networking...
==> myvm: Checking if box 'centos/7' version '1905.1' is up to date...
==> myvm: Setting the name of the VM: demo-05_myvm_1562573100554_42608
==> myvm: Clearing any previously set network interfaces...
==> myvm: Preparing network interfaces based on configuration...
    myvm: Adapter 1: nat
==> myvm: Forwarding ports...
    myvm: 22 (guest) => 2222 (host) (adapter 1)
==> myvm: Booting VM...
==> myvm: Waiting for machine to boot. This may take a few minutes...
    myvm: SSH address: 127.0.0.1:2222
    myvm: SSH username: vagrant
    myvm: SSH auth method: private key
    myvm: Warning: Connection reset. Retrying...
    myvm:
    myvm: Vagrant insecure key detected. Vagrant will automatically replace
    myvm: this with a newly generated keypair for better security.
    myvm:
    myvm: Inserting generated public key within guest...
    myvm: Removing insecure key from the guest if it's present...
    myvm: Key inserted! Disconnecting and reconnecting using new SSH key...
==> myvm: Machine booted and ready!
[myvm] No Virtualbox Guest Additions installation found.
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror01.idc.hinet.net
 * extras: mirror01.idc.hinet.net
 * updates: mirror01.idc.hinet.net
Package binutils-2.27-34.base.el7.x86_64 already installed and latest version
Package 1:make-3.82-23.el7.x86_64 already installed and latest version
Package bzip2-1.0.6-13.el7.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package gcc.x86_64 0:4.8.5-36.el7_6.2 will be installed
......  略
Installing Virtualbox Guest Additions 6.0.8 - guest version is unknown
Verifying archive integrity... All good.
Uncompressing VirtualBox 6.0.8 Guest Additions for Linux........
VirtualBox Guest Additions installer
Copying additional installer modules ...
Installing additional modules ...
VirtualBox Guest Additions: Starting.
VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel
modules.  This may take a while.
VirtualBox Guest Additions: To build modules for other installed kernels, run
VirtualBox Guest Additions:   /sbin/rcvboxadd quicksetup <version>
VirtualBox Guest Additions: or
VirtualBox Guest Additions:   /sbin/rcvboxadd quicksetup all
VirtualBox Guest Additions: Building the modules for kernel
3.10.0-957.12.2.el7.x86_64.
Redirecting to /bin/systemctl start vboxadd.service
Redirecting to /bin/systemctl start vboxadd-service.service
Unmounting Virtualbox Guest Additions ISO from: /mnt
==> myvm: Checking for guest additions in VM...
==> myvm: Rsyncing folder: /vagrant/demo-05/ => /vagrant
==> myvm: Mounting shared folders...
    myvm: /tmp/github => /github
```
這樣你在 VＭ 內的 /tmp/github 可以找到外部實體機的 /github 資料內容了  
但缺點是會多花點時間在幫 VM 安裝 Virtualbox Guest Additions   

## 客製化
上面的 Vagrantfile 啟動的虛擬機 預設是 1 cpu 跟 512 MB 記憶體, 有的時候比較重的應用是跑不動的, 我先說明怎麼去客製化調整這些東西.  

看這個 Vagrantfile 範例檔
``` Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :myvm do |myvm|
    myvm.vm.box = "centos/7"
    myvm.vm.hostname = 'vm001'
    
    # 增加 hostonly 網路, 並指定 IP
    myvm.vm.network :private_network, ip: "192.168.56.10"
    
    myvm.vm.provider :virtualbox do |vb|
      # 這邊設定虛擬機名稱，不指定的話會用亂數
      vb.name = "ansible-controller"
      # 指定使用 cpu 核數
      vb.cpus = 2
      # 指定記憶體大小
      vb.memory = 1024
      # 通常是不用 gui 需要的話可以開
      vb.gui = false
      # 讓虛擬機可使用實體機 DNS 加快啟動速度
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
    
    # 第一次啟動時執行 shellscript
    # 預設是不允許密碼登入, 這邊修改掉可以用密碼登入
    myvm.vm.provision :shell, inline: <<-SHELL
	  	sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
	  	systemctl restart sshd
	  SHELL
  end
end
```
啟動後 預設的帳密為  
root / vagrant  
vagrant / vagrant  

這樣就可以用帳號密碼登入操作了  

## 多台
我找到一個範例還蠻好用的  
``` Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

# 指定 Box
BOX_IMAGE = "centos/7"

cluster = {
  "master" => { :ip => "192.168.56.101", :cpus => 1, :mem => 1024 },
  "slave" => { :ip => "192.168.56.102", :cpus => 1, :mem => 1024 }
}

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  
  cluster.each_with_index do |(hostname, info), index|
    config.vm.define hostname do |cfg|
      cfg.vm.provider :virtualbox do |vb, override|
        config.vm.box = BOX_IMAGE
        override.vm.network :private_network, ip: "#{info[:ip]}"
        override.vm.hostname = hostname
        vb.name = hostname
        vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpus], "--hwvirtex", "on"]
      end # end provider
    end # end config
  end # end cluster
end
```
這個可以一次配置 IP 跟機器  

另一種就比較簡易循序開幾台相同的而已  
``` Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

# 指定 Box
BOX_IMAGE = "centos/7"

NODE_COUNT = 4

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  NODE_COUNT.times do |i|
    node_id = "node#{i}"
    config.vm.define node_id do |node|
      node.vm.box = BOX_IMAGE
      node.vm.hostname = "#{node_id}.intranet.local"
      node.vm.network :private_network, ip: "192.168.56.#{10+i}"
    end
  end
  
end
```

## References
[VirtualBox 虚拟机里网络很慢的解决方法](https://www.pylist.com/t/1555072754)  
[Vagrant: Slow internet connection in guest](https://serverfault.com/questions/495914/vagrant-slow-internet-connection-in-guest/496612#496612)  
[Day 26 Vagrant 構建及配置虛擬環境](https://ithelp.ithome.com.tw/articles/10205780)  
[Vagrant 學習筆記](https://vagrantpi.github.io/2018/02/11/vagrant/)  
[Vagrant Tutorial – From Nothing To Multi-Machine](https://manski.net/2016/09/vagrant-multi-machine-tutorial/)  
[Vagrant 安裝說明 - William Yeh](https://school.soft-arch.net/blog/1061/install-vagrant)  
[使用Vagrant跟Docker快速建構開發與測試環境](http://samchu.logdown.com/posts/288889-docker-fast-with-vagrant-and-construct-development-and-test-environments)  
[roblayton/Vagrantfile](https://gist.github.com/roblayton/c629683ca74658412487)  
[Dynamic multi-machine Vagrantfile](https://maci0.wordpress.com/2013/11/09/dynamic-multi-machine-vagrantfile/)  

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。