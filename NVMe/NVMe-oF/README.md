# NVMe-oF

- [NVMe-oF](#nvme-of)
  - [一些事实](#一些事实)
    - [性能](#性能)
  - [在Ubuntu-24.04虚拟机的部署](#在ubuntu-2404虚拟机的部署)
    - [Target端](#target端)
      - [环境准备](#环境准备)
      - [内核模块加载](#内核模块加载)
      - [配置Target](#配置target)
        - [创建子系统](#创建子系统)
        - [创建命名空间](#创建命名空间)
        - [创建端口](#创建端口)
      - [验证Target配置状态](#验证target配置状态)
    - [部署Host端](#部署host端)
    - [Target持久化配置](#target持久化配置)
  - [物理机NVMe-oF TCP P2P实验](#物理机nvme-of-tcp-p2p实验)

## 一些事实

### 性能

Target端纯粹消耗CPU算力，哪怕网卡具备RDMA支持，对于CPU的资源消耗依然非常严重，因为想要实现通用储存介质的数据写入，必须经过CPU计算（示例路径：Host -(40G光纤)-> Target网卡 -> VFS -> 数据落盘），因此高负载下CPU资源占用将会非常高。

与之对应的，Host端的数据也必须经过CPU，不可能实现从EP端直接送往RDMA网卡，这样就不能实现纯粹PCIe路径上的数据运输了。

## 在Ubuntu-24.04虚拟机的部署

### Target端

#### 环境准备

```Shell
sudo apt install nvme-cli -y
```

#### 内核模块加载

```Shell
sudo modprobe nvmet
sudo modprobe nvmet-tcp
```

该内核模块位于`/lib/modules/<内核版本>/kernel/drivers/nvme`目录中，在X64架构上有如下模块：

```txt
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/target
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/target/nvmet-pci-epf.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/target/nvmet-fc.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/target/nvmet-rdma.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/target/nvmet.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/target/nvmet-tcp.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/target/nvme-loop.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/common
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/common/nvme-auth.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/common/nvme-keyring.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/host
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/host/nvme.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/host/nvme-rdma.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/host/nvme-fabrics.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/host/nvme-tcp.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/host/nvme-fc.ko.zst
/lib/modules/6.17.0-40-generic/kernel/drivers/nvme/host/nvme-core.ko.zst
```

其他的类似`nvmet-rdma`或者`nvmet-fc`模块，在有40G网卡的时候可能也是需要的，但是更重要的是要下载对应网卡的驱动程序，如`MLNX_OFED_LINUX`驱动。

#### 配置Target

> [!WARNING]
> 配置Target的所有步骤必须执行`sudo su`进入`root`用户执行。

##### 创建子系统

进入配置目录：

```Shell
cd /sys/kernel/config/nvmet/
```

创建子系统：

```Shell
mkdir subsystems/nqn.2026-07.com.example:target1
cd subsystems/nqn.2026-07.com.example:target1
```

设置所有客户端均可连接：

```Shell
echo 1 > attr_allow_any_host
```

##### 创建命名空间

```Shell
mkdir /sys/kernel/config/nvmet/namespaces/1
cd /sys/kernel/config/nvmet/namespaces/1
```

绑定设备（这里的设备可以是任何块设备文件或者普通文件，包括maadm创建的RAID阵列、一个.img数据文件等特殊文件）：

```Shell
echo /dev/nvme0n1 > device_path
```

启用该命名空间：

```Shell
echo 1 > enable
```

##### 创建端口

进入到端口创建目录：

```Shell
mkdir /sys/kernel/config/nvmet/ports/1
cd /sys/kernel/config/nvmet/ports/1
```

进行如下设置：

```Shell
echo <本机IP地址> | sudo tee addr_traaddr
echo 4420 | sudo tee addr_trsvcid
echo tcp | sudo tee addr_trtype
echo ipv4 | sudo tee addr_adrfam
```

将创建的端口和子系统关联起来，使配置生效：

```Shell
ln -s /sys/kernel/config/nvmet/subsystems/nqn.2026-07.com.example:target1 \
/sys/kernel/config/nvmet/ports/1/subsystems/nqn.2026-07.com.example:target1
```

#### 验证Target配置状态

```Shell
sudo dmesg | grep -i nvmet
```

### 部署Host端

Host端同样需要进行[环境准备](#环境准备)和[内核模块加载](#内核模块加载)。

发现Target：

```Shell
sudo nvme discover -t tcp -a <Target IP> -s 4420
```

连接到Target：

```Shell
sudo nvme connect -t tcp -n nqn.2026-07.com.example:target1 -a <Target IP> -s 4420
```

### Target持久化配置

新建`/usr/local/bin/setup_nvmeof_target.sh`，并写入如下内容：

```txt
#!/bin/bash

modprobe nvmet
modprobe nvmet-tcp

cd /sys/kernel/config/nvmet/
mkdir subsystems/nqn.2026-07.com.example:target1
cd subsystems/nqn.2026-07.com.example:target1
echo 1 > attr_allow_any_host

mkdir /sys/kernel/config/nvmet/subsystems/nqn.2026-07.com.example:target1/namespaces/1
cd /sys/kernel/config/nvmet/subsystems/nqn.2026-07.com.example:target1/namespaces/1
echo 【设备路径】 > device_path
echo 1 > enable

mkdir /sys/kernel/config/nvmet/ports/1
cd /sys/kernel/config/nvmet/ports/1
echo 【本机IP】 > addr_traddr
echo 4420 > addr_trsvcid
echo tcp > addr_trtype
echo ipv4 > addr_adrfam

ln -s /sys/kernel/config/nvmet/subsystems/nqn.2026-07.com.example:target1 \
/sys/kernel/config/nvmet/ports/1/subsystems/nqn.2026-07.com.example:target1

dmesg | grep -i nvmet
```

新建`/etc/systemd/system/nvmeof-target.service`，并写入如下内容：

```txt
[Unit]
Description=NVMe-oF Target Setup
# 关键：确保网络已完全就绪后再启动，避免绑定端口失败[reference:7][reference:8]
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
# 脚本只需执行一次
RemainAfterExit=yes
# 指定要执行的脚本
ExecStart=/usr/local/bin/setup_nvmeof_target.sh

[Install]
# 将服务挂钩到多用户模式，实现开机启动
WantedBy=multi-user.target
```

启用服务：

```Shell
sudo systemctl daemon-reload
sudo systemctl enable nvmeof-target.service
```

重启之后，验证服务状态：

```Shell
sudo systemctl status nvmeof-target.service
```

如果服务成功启动，将会有如下内容：

```txt
● nvmeof-target.service - NVMe-oF Target Setup
     Loaded: loaded (/etc/systemd/system/nvmeof-target.service; enabled; preset: enabled)
     Active: active (exited) since Tue 2026-07-14 17:00:12 CST; 59s ago
    Process: 1761 ExecStart=/usr/local/bin/setup_nvmeof_target.sh (code=exited, status=0/SUCCESS)
   Main PID: 1761 (code=exited, status=0/SUCCESS)
        CPU: 40ms

7月 14 17:00:12 ubuntu systemd[1]: Starting nvmeof-target.service - NVMe-oF Target Setup...
7月 14 17:00:12 ubuntu setup_nvmeof_target.sh[1806]: [   10.013191] nvmet: adding nsid 1 to subsystem nqn.2026-07.com.example:target1
7月 14 17:00:12 ubuntu setup_nvmeof_target.sh[1806]: [   10.014940] nvmet_tcp: enabling port 1 (192.168.18.133:4420)
7月 14 17:00:12 ubuntu systemd[1]: Finished nvmeof-target.service - NVMe-oF Target Setup.
```

## 物理机NVMe-oF TCP P2P实验
