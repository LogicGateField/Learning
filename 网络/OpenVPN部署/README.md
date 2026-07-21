# OpenVPN部署

- [OpenVPN部署](#openvpn部署)
  - [一键安装](#一键安装)
    - [服务端附加配置](#服务端附加配置)
  - [客户端](#客户端)
    - [在服务器生成客户端配置文件](#在服务器生成客户端配置文件)
    - [客户端systemd服务部署](#客户端systemd服务部署)
  - [附录](#附录)
    - [修改VPN网段](#修改vpn网段)
    - [回车符不匹配问题](#回车符不匹配问题)
    - [重新注册一个已经删除的客户端](#重新注册一个已经删除的客户端)

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

### 服务端附加配置

> [!WARNING]
> 这个配置是为了让客户端能够正常访问网络！

编辑服务端的`/etc/openvpn/server.conf`文件，找到如下几行：

```txt
push "dhcp-option DNS 94.140.14.14"
push "dhcp-option DNS 94.140.15.15"
push "redirect-gateway def1 bypass-dhcp"
```

修改为（注意替换`10.8.0.0`为实际的网段）：

```txt
push "dhcp-option DNS 114.114.114.114"
# push "dhcp-option DNS 94.140.15.15"
# push "redirect-gateway def1 bypass-dhcp"
push "route 10.8.0.0 255.255.255.0 vpn_gateway"
```

修改完成后重启服务器即可。

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

### 重新注册一个已经删除的客户端

如果直接执行`openvpn-install.sh`脚本注册一个之前删除的客户端同名的客户端，就会收到以下失败信息：

```Shell
指定的客户端 CN 已在 easy-rsa 中找到，请选择另一个名称。
```

这个情况涉及RSA证书的问题。需要编辑`/etc/openvpn/easy-rsa/pki/index.txt`文件，并且删除（或者注释掉）含有客户端名称的那一行，然后重新执行`openvpn-install.sh`脚本进行客户端注册就行了。
