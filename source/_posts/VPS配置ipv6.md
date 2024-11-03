---
title: VPS配置ipv6
date: 2024-11-03 19:16:26
description: 最近好几个VPS的ipv6都出现了问题，来来回回调节了很久，把配置过程记录一下
tags: [网络]
categories: 
- [网络]
---

## 一、开启或关闭ipv6

&emsp;&emsp;配置这个的原因是某台VPS的商家出了问题，ipv6不可用了，但是VPS里面的ipv6配置还在。在访问某些网址的时候，如果解析的ip是ipv6的话反而会出现访问不正常的情况。所以把ipv6功能关闭是一个不错的选择。

&emsp;&emsp;VPS的操作系统是debin12，我选择修改 /etc/sysctl.conf 文件来禁用 IPv6：

&emsp;&emsp;1、编辑 /etc/sysctl.conf 文件：

```
sudo nano /etc/sysctl.conf
```

&emsp;&emsp;2、添加或修改原始内容如下：

```bash
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

&emsp;&emsp;ctrl x 加 y保存退出。

&emsp;&emsp;3、应用更改：

```
sudo sysctl -p
```

&emsp;&emsp;4、查看是否生效：

```bash
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
```

&emsp;&emsp;如果输出为 1，则表示 ipv6 已被禁用。反过来，如果要重新启用ipv6，就把sysctl.conf 里面的选项改成0（不是删除掉），然后重新应用一下。

## 二、使用Netplan配置ipv6

&emsp;&emsp;第二台VPS是一台Ubuntu22.04的机子，商家给了ipv6地址，但VPS实际没有配置上，需要手动添加。这里选择使用Netplan配置ipv6。当然前提是ipv6功能需要启动（参考前面）：

&emsp;&emsp;1、编辑Netplan配置文件：

```
sudo nano /etc/netplan/00-installer-config.yaml
```

&emsp;&emsp;不同商家这个文件的文件名可能会不同，可以先进/etc/netplan目录看一下配置文件的名称。

&emsp;&emsp;2、添加或修改ipv6配置（ip只是示例）：

```
network:
  version: 2
  ethernets:
    eth0:
      dhcp6: no
      addresses:
        - 2001:db8::2/64
      routes:
        - to: ::/0
          via: 2001:db8::1
      nameservers:
        addresses:
          - 2001:4860:4860::8888
          - 2001:4860:4860::8844
```

&emsp;&emsp;这里需要知道商家提供的ipv6地址，子网掩码和网关信息。其中 addresses-2001:db8::2 代表IP地址，/64 代表子网掩码，routes-2001:db8::1 代表网关，需要和ip在子网掩码对应的同一个网段内（如果不对应说明商家给的配置信息有问题）。2001:4860:4860::8888 和2001:4860:4860::8844 是谷歌的DNS，可以按照自己的需求添加别的DNS服务器。

&emsp;&emsp;3、验证配置文件语法：

```bash
sudo netplan try
```

&emsp;&emsp;4、应用配置：

```
sudo netplan apply
```

&emsp;&emsp;5、验证配置：

```bash
# 显示网络配置
ifconfig

# 测试连接
ping -6 google.com
```

## 三、使用interfaces文件配置ipv6

&emsp;&emsp;非常不凑巧的事情是第三台VPS又出问题了，这回是商家给错了ip地址，和网关不在一个网段内，导致无法联网。更新地址后VPS没有自动更新，所以又要重新配置一下。这台VPS的系统是Debian 11。但由于没有安装Netplan，这里用更原始的interfaces文件配置ipv6。

&emsp;&emsp;1、同样是编辑网络配置文件：

```
sudo nano /etc/network/interfaces
```

&emsp;&emsp;不过这里我的interfaces是include了interfaces.d目录，所以我还得去interfaces/interfaces.d目录里找一下被引用文件的名字，然后进行编辑。

&emsp;&emsp;2、添加或修改配置：

```bash
iface eth0 inet6 static
    address 2001:db8::2/64         
    gateway 2001:db8::1             
    dns-nameservers 2001:4860:4860::8888 2001:4860:4860::8844
```

&emsp;&emsp;和前面一样，都是添加ip地址，子网掩码和网关，还有DNS服务器

&emsp;&emsp;3、应用配置：

```
sudo systemctl restart networking
```

&emsp;&emsp;当然，即使进行了多次配置，还是有一台VPS始终连接不上网络，这种情况就只能找商家解决了。
