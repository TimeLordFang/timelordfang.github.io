---
layout: post
title: "OpenNebula 安装 配置 "
date: 2016-03-09
tags: [opennebula, kvm , ruby , linux]
---



opennebula安装
===

### 0x01 配置yum源

前端控制接节点和后端计算节点，皆配置如下源
```shell
yum install -y epel-release
cat /etc/yum.repos.d/opennebula.repo
[opennebula]
name=opennebula
baseurl=http://downloads.opennebula.org/repo/4.14/CentOS/6/x86_64/
enabled=1
gpgcheck=0
```

### 0x02 安装控制端

#### 安装核心服务以及管理UI
```shell
yum install opennebula-server opennebula-sunstone -y
```

#### 安装ruby等依赖
```shell
/usr/share/one/install_gems 
```
基本上默认回车即可，实测最后有几个组件装不上，但似乎没有影响服务启动运行。

#### 相关配置并开启服务
修改`/etc/one/sunstone-server.conf `，将`:host 127.0.0.1`改为实际使用的ip，需要修改db的话在/etc/one/oned.conf里。

```shell
/etc/init.d/opennebula start
/etc/init.d/opennebula-sunstone start 
```
实测按照书本最后起不来opennebula-sunstone，看日志可能有个ruby依赖，命令`gem install builder`，装上就可以了。

#### 配置ssh认证
```shell
su - oneadmin
cat ~/.ssh/config   
Host *
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
```

#### 修改数据传输驱动以及存储路径

`onedatastore list`命令可查看，前两个默认使用share，需要改成ssh，这样可以使用每台机子自己的存储，只是在新建的时候需要使用ssh传送镜像，新建速度慢一点点而已。
ruby定义使用的编辑器路径是`/usr/bin/vi`，所以要先链接一下。然后用onedatastore命令更新配置。
```shell
ln -s /usr/bin/vim /usr/bin/vi 
onedatastore update 0
onedatastore update 1
onedatastore update 2
```
TM_MAD对应项改成ssh，BASE_PATH对应项改成自己定义路径，记得把路径改成对应权限。/etc/one/oned.conf里面的DATASTORE_LOCATION 项最好也写上对应的存储路径。

web登陆账号密码在/var/lib/one/.one/one_auth
直接编辑似乎会出问题
http://serverfault.com/questions/545108/opennebula-sunston-user-oneadmin-password


### 0x03 安装kvm计算节点

```shell
yum install opennebula-node-kvm -y 
rm -f /etc/libvirtd/qemu/networks/autostart/*
/etc/init.d/libvirtd start
/etc/init.d/messagebus start 
```
copy opennebula-server sshkey

