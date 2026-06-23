# VMware使用技巧

- [VMware使用技巧](#vmware使用技巧)
  - [共享文件夹在Ubuntu下的使用](#共享文件夹在ubuntu下的使用)
  - [Linux虚拟机硬盘压缩](#linux虚拟机硬盘压缩)
  - [键盘鼠标响应不灵敏、有延迟](#键盘鼠标响应不灵敏有延迟)

## 共享文件夹在Ubuntu下的使用

完成VMware软件的共享文件夹添加之后，打开Ubuntu虚拟机，在终端中编辑`/etc/fstab`文件：

```Shell
sudo gedit /etc/fstab
```

添加如下内容：

```txt
.host:/ /mnt/hgfs fuse.vmhgfs-fuse allow_other,defaults 0 0
```

然后保存退出，在终端中创建`/mnt/hgfs`目录：

```Shell
sudo mkdir /mnt/hgfs
```

之后重启虚拟机，就可以在`/mnt/hgfs`目录中找到添加的共享文件夹了。

## Linux虚拟机硬盘压缩

> https://blog.csdn.net/qq_35358125/article/details/107081122

例如，在虚拟机中设置了两个虚拟挂载点，分别是`/`和`/data`，就可以执行以下指令一次性压缩这两个虚拟挂载点：

```Shell
sudo vmware-toolbox-cmd disk wipe /; sudo vmware-toolbox-cmd disk wipe /data; sudo vmware-toolbox-cmd shrinkonly
```

## 键盘鼠标响应不灵敏、有延迟

> https://www.cnblogs.com/bfmhno3/p/18893864

在虚拟机的`*.vmx`文件中添加如下行：

```txt
keyboard.vusb.enable = "TRUE"
```
