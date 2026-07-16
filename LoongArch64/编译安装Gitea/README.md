# 编译安装Gitea

- [编译安装Gitea](#编译安装gitea)
  - [环境与编译工具部署](#环境与编译工具部署)
  - [编译方法](#编译方法)
  - [清理方法](#清理方法)
  - [注册为服务](#注册为服务)
  - [附录](#附录)

## 环境与编译工具部署

本次编译平台为X64的Ubuntu-24.04虚拟机，使用以下指令安装编译工具：

```Shell
sudo apt install npm go make cmake -y
```

实测，Ubuntu-24.04默认的这些工具版本已经够用了。

## 编译方法

按照[Gitea官方编译说明][1]进行编译。注意最后一步编译的时候替换成：

```Shell
GOOS=linux GOARCH=loong64 TAGS="bindata sqlite sqlite_unlock_notify" make build
```

编译完成后，观察得到的文件的信息：

```Shell
ubuntu@ubuntu:/data/gitea$ file ./gitea
./gitea: ELF 64-bit LSB executable, LoongArch, version 1 (SYSV), statically linked, Go BuildID=lzpaNU7zjtoQyaMFlIv-/eM3PbRQ7dFb60w63YaTx/E26WgwFYI6Afbx1TUwN1/TIKNCUz65RMbOdxf8IAR, stripped
```

确实是龙芯架构的可执行文件。

在X64平台上使用`qemu-loong64 ./gitea`可以成功启动该编译结果。

## 清理方法

执行以下命令清理所有中间结果和编译结果：

```Shell
make clean
```

## 注册为服务

添加用户`git`，用于操作隔离：

```Shell

```

创建文件`/etc/systemd/system/gitea.service`，并写入以下内容：

```txt
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target
###
# Don't forget to add the database service dependencies
###
#
#Wants=mysql.service
#After=mysql.service
#
#Wants=mariadb.service
#After=mariadb.service
#
#Wants=postgresql.service
#After=postgresql.service
#
#Wants=memcached.service
#After=memcached.service
#
#Wants=redis.service
#After=redis.service
#
###
# If using socket activation for main http/s
###
#
#After=gitea.main.socket
#Requires=gitea.main.socket
#
###
# (You can also provide gitea an http fallback and/or ssh socket too)
#
# An example of /etc/systemd/system/gitea.main.socket
###
##
## [Unit]
## Description=Gitea Web Socket
## PartOf=gitea.service
##
## [Socket]
## Service=gitea.service
## ListenStream=<some_port>
## NoDelay=true
##
## [Install]
## WantedBy=sockets.target
##
###

[Service]
# Uncomment the next line if you have repos with lots of files and get a HTTP 500 error because of that
# LimitNOFILE=524288:524288
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
# If using Unix socket: tells systemd to create the /run/gitea folder, which will contain the gitea.sock file
# (manually creating /run/gitea doesn't work, because it would not persist across reboots)
#RuntimeDirectory=gitea
ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea
# If you install Git to directory prefix other than default PATH (which happens
# for example if you install other versions of Git side-to-side with
# distribution version), uncomment below line and add that prefix to PATH
# Don't forget to place git-lfs binary on the PATH below if you want to enable
# Git LFS support
#Environment=PATH=/path/to/git/bin:/bin:/sbin:/usr/bin:/usr/sbin
# If you want to bind Gitea to a port below 1024, uncomment
# the two values below, or use socket activation to pass Gitea its ports as above
###
#CapabilityBoundingSet=CAP_NET_BIND_SERVICE
#AmbientCapabilities=CAP_NET_BIND_SERVICE
###
# In some cases, when using CapabilityBoundingSet and AmbientCapabilities option, you may want to
# set the following value to false to allow capabilities to be applied on gitea process. The following
# value if set to true sandboxes gitea service and prevent any processes from running with privileges
# in the host user namespace.
###
#PrivateUsers=false
###

[Install]
WantedBy=multi-user.target
```

注意文本中的路径替换。这里可以全部替换为`/data/git_server`，或者其他文件夹。

一般来说，将所有数据与配置放在一个文件夹里面，有利于数据迁移和备份。

## 附录

[1]: https://docs.gitea.com/zh-cn/1.19/installation/install-from-source
