# 编译安装Gitea

- [编译安装Gitea](#编译安装gitea)
  - [编译方法](#编译方法)
  - [附录](#附录)

## 编译方法

在X64平台上（测试使用的是Ubuntu-24.04虚拟机）安装go、npm等工具，然后按照[Gitea官方编译说明][1]进行编译。注意最后一步编译的时候替换成：

```Shell
GOOS=linux GOARCH=loong64 TAGS="bindata sqlite sqlite_unlock_notify" make build
```

编译完成后，在X64平台上使用`qemu-loong64 ./gitea`可以成功启动该编译结果。

## 附录

[1]: https://docs.gitea.com/zh-cn/1.19/installation/install-from-source
