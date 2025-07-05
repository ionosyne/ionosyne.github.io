---
title: 在VMware虚拟机中设置代理
date: 2024-09-01 10:13:51
description: 配置VMware虚拟机，使其能够使用主机的代理
tags: [VMware]
categories: 
- [网络]
---

## 一、写在前面

&emsp;&emsp;最近需要在无网络Linux环境下安装python环境，也查了很久的资料，发现使用conda打包环境再整体迁移是比较简便的方法。使用conda打包的前提是python环境里所有包都是使用conda install安装的。但国内访问conda源一直不太稳定，而阿里、清华的源安装总会出现各种各样的问题，所以使用代理是不错的选择。那么问题来了，负责打包的Linux环境是配置在VMware中的，该如何使用代理呢。

## 二、环境配置

&emsp;&emsp;我没有选择在虚拟机里安装代理软件，而是使用主机的代理。主机的代理软件需要开启允许来自局域网的连接，并且记下主机的局域网ip和代理的端口号。例如我的主机ip是192.168.8.192，http和socks5的端口号分别是10809和10808。

&emsp;&emsp;为了虚拟机能访问主机网络，需要配置虚拟机ip和主机ip位于同一网段。最简单的设置方式是将虚拟机的网络设置成桥接模式，这样就不用专门去配置ip地址了。

<p align="center">
    <img src="https://p.iz.mk/i/2025/07/05/6868ba6c75d92.webp" style="zoom:50%;" />
</p>


&emsp;&emsp;网络设置好以后进入虚拟机。我使用的是centos7，进入之后在右上角的网络设置network-network proxy里面把主机ip和代理端口填进去，就配置好了。

<p align="center">
    <img src="https://p.iz.mk/i/2025/07/05/6868bab04fc7d.webp" style="zoom:70%;" />
</p>


## 三、简单测试

&emsp;&emsp;由于只配置了http和socks代理，直接ping还是无法ping通国外ip的，这里用cURL做简单测试：

<p align="center">
    <img src="https://p.iz.mk/i/2025/07/05/6868bac674c52.webp" style="zoom:70%;" />
</p>


&emsp;&emsp;发现有数据返回，说明配置成功。接下来就可以正常使用conda安装所需环境了。
