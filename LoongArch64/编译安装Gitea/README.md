# 编译安装Gitea

- [编译安装Gitea](#编译安装gitea)
  - [环境与编译工具部署](#环境与编译工具部署)
  - [编译方法](#编译方法)
  - [清理方法](#清理方法)
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

## 附录

[1]: https://docs.gitea.com/zh-cn/1.19/installation/install-from-source
