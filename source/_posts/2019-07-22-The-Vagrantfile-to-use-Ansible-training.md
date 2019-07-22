---
title: The Vagrantfile to use Ansible training
tags:
  - ansible
  - vagrant
categories:
  - DevOps
date: 2019-07-22 18:38:54
---


The Vagrantfile training to use Ansible

<!--more-->

Vagrantfile
``` Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

NODE_COUNT = 2

Vagrant.configure("2") do |config|
  # Don't auto update VirtualBox Guest Additions
  config.vbguest.auto_update = false

  config.vm.define "myvm" do |myvm|
    myvm.vm.box = "centos/7"
    myvm.vm.network :private_network, ip: "192.168.56.10"
#    myvm.vm.synced_folder "/github", "/tmp/github"
    myvm.vm.provision :shell, inline: <<-SHELL
	  	sudo yum install epel-release -y
      sudo yum update -y
      sudo yum install python-pip -y
      sudo pip install --upgrade pip
      sudo yum install ansible -y
	  SHELL
  end

  NODE_COUNT.times do |i|
    node_id = "node#{i}"
    config.vm.define node_id do |node|
      node.vm.box = "centos/7"
      node.vm.hostname = "#{node_id}.intranet.local"
      node.vm.network :private_network, ip: "192.168.56.#{20+i}"
    end
  end

end
```

After run vagrant up

![https://i.imgur.com/nDa51ID.png](https://i.imgur.com/nDa51ID.png)

``` bash
$ vagrant up
Bringing machine 'myvm' up with 'virtualbox' provider...
Bringing machine 'node0' up with 'virtualbox' provider...
Bringing machine 'node1' up with 'virtualbox' provider...
==> myvm: Importing base box 'centos/7'...
==> myvm: Matching MAC address for NAT networking...
.
.
.
==> node1: Setting hostname...
==> node1: Configuring and enabling network interfaces...
==> node1: Rsyncing folder: /vagrant/demo-06/ => /vagrant
zhushanglide-MacBook-Air:demo-06 shanglichu$ vagrant ssh myvm
[vagrant@localhost ~]$ sudo ansible --version
ansible 2.8.1
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.5 (default, Jun 20 2019, 20:27:34) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
[vagrant@localhost ~]$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85887sec preferred_lft 85887sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:7b:0e:3c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.10/24 brd 192.168.56.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe7b:e3c/64 scope link
       valid_lft forever preferred_lft forever
```

you can see myvm has ansible and fixed IP.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。

