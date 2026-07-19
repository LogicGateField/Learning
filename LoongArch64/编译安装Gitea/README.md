# 编译安装Gitea

- [编译安装Gitea](#编译安装gitea)
  - [环境与编译工具部署](#环境与编译工具部署)
    - [X86\_64，Ubuntu-24.04](#x86_64ubuntu-2404)
    - [LoongArch，OpenEuler-24.03-LTS](#loongarchopeneuler-2403-lts)
  - [编译方法](#编译方法)
  - [清理方法](#清理方法)
  - [注册为服务](#注册为服务)
  - [附录](#附录)
    - [OpenEuler系统上的防火墙设置](#openeuler系统上的防火墙设置)

## 环境与编译工具部署

### X86_64，Ubuntu-24.04

编译平台为X64的Ubuntu-24.04虚拟机，使用以下指令安装编译工具：

```Shell
sudo apt install npm golang make cmake -y
```

### LoongArch，OpenEuler-24.03-LTS

编译平台为LoongArch（Loong64）物理机，CPU为3A6000，使用以下指令部署编译工具：

```Shell
dnf install -y npm go make cmake
```

## 编译方法

> [!WARNING]
> 不要在X64平台上使用交叉编译器编译，如果要编译龙芯架构的就在龙芯架构的QEMU虚拟机或者物理机上进行编译，ARM64同理。在X64平台上进行交叉编译有可能会导致SQLite3无法使用！

按照[Gitea官方编译说明][1]进行编译。注意最后一步编译的时候替换成：

```Shell
TAGS="bindata sqlite sqlite_unlock_notify" make build
```

## 清理方法

执行以下命令清理所有中间结果和编译结果：

```Shell
make clean
```

## 注册为服务

添加用户`git`，用于操作隔离（密码啥的也可以设置）：

```Shell
adduser git
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

> [!WARNING]
> 整一个包含运行文件和配置文件的目录（这里的示例是`/data/git_server`）必须归属于`git`用户。在`root`中使用`chown git /xx/xxx`命令可以实现目录的归属设置。

## 附录

### OpenEuler系统上的防火墙设置

如果在OpenEuler系统上部署Gitea服务器，则必须将服务器对外服务的端口的TCP和UDP协议加入到防火墙的入站规则中：

```Shell
firewall-cmd --zone=public --add-port=3000/tcp --permanent
firewall-cmd --zone=public --add-port=3000/udp --permanent
firewall-cmd --reload
```

[1]: https://docs.gitea.com/zh-cn/1.19/installation/install-from-source
