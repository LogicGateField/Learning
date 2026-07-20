# OpenVPN部署

- [OpenVPN部署](#openvpn部署)
  - [一键安装](#一键安装)
  - [客户端](#客户端)
    - [在服务器生成客户端配置文件](#在服务器生成客户端配置文件)
    - [客户端systemd服务部署](#客户端systemd服务部署)
    - [客户端附加配置](#客户端附加配置)
  - [附录](#附录)
    - [修改VPN网段](#修改vpn网段)
    - [回车符不匹配问题](#回车符不匹配问题)

## 一键安装

在主目录下执行：

```Shell
wget https://raw.githubusercontent.com/nas-tool/openvpn-install/master/openvpn-install.sh
chmod +x openvpn-install.sh && ./openvpn-install.sh
```

按提示填写信息就行。安装完不要删除这个脚本，后面还有用的。

执行`ip a`，有以下网卡信息则表明配置成功：

```Shell
tun0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none 
    inet 10.9.0.1/24 brd 10.8.0.255 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::64cb:115e:493a:88bf/64 scope link stable-privacy proto kernel_ll 
       valid_lft forever preferred_lft forever
```

> [!WARNING]
> 配置完OpenVPN之后，记得去云服务器提供商的安全组放行1149端口的UDP数据包，否则无法连接

## 客户端

### 在服务器生成客户端配置文件

在主目录运行：

```Shell
./openvpn-install.sh
```

按提示输入信息就行。

### 客户端systemd服务部署

以下是指令模板：

```Shell
cp ./<配置文件> /etc/openvpn/client/<VPN名称>.conf
systemctl enable --now openvpn-client@<VPN名称>
```

例如，现在我在主目录有一个`zflab_nas_b0.ovpn`配置文件，我需要创建一个名字是`vpn_b0`的VPN虚拟网卡，我可以执行以下命令：

```Shell
cp ./zflab_nas_b0.ovpn /etc/openvpn/client/vpn_b0.conf
systemctl enable --now openvpn-client@vpn_b0
```

### 客户端附加配置

客户端的`.ovpn`文件需要加上以下两行（注意替换`10.8.0.0`为实际的网段）：

> [!WARNING]
> 最好加上这两行，不然网络有可能会出问题哦

```txt
pull-filter ignore redirect-gateway
route 10.8.0.0 255.255.255.0 vpn_gateway
```

## 附录

### 修改VPN网段

编辑`openvpn-install.sh`文件，将所有`10.8.0.`替换成`10.9.0.`（示例，需要什么网段都可以自己更改）即可。

以下是一个`sed`命令示例（当然也可以用VS Code改）：

```Shell
sed -i 's/10\.8\.0\./10\.9\.0\./g' openvpn-install.sh
```

### 回车符不匹配问题

如果执行过程中出现以下错误信息：

```Shell
-bash: ./openvpn-install.sh: /bin/bash^M: bad interpreter: No such file or directory
```

则执行以下命令：

```Shell
sed -i 's/\r$//' ./openvpn-install.sh
```
