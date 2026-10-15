---
title: realm的安装和配置
date: 2026-10-15 22:53:24
description: realm的安装和配置过程记录，老是忘记，做个备忘
tags: [realm]
categories: 
- [网络]
---

## 一、下载可执行程序

&emsp;&emsp;realm 的发布页面在 [Github](https://github.com/zhboner/realm/releases) ，根据自己的系统下载最新版本即可。比如VPS用64位的 Debian 系统，就下载 realm-x86_64-unknown-linux-gnu.tar.gz。

&emsp;&emsp;然后在本地解压，得到一个名为 realm 的可执行程序。把这个文件用 SFTP 上传到 VPS 的 /root 目录。用 ssh 连接到 VPS 后，给 realm 加上可执行权限：

```bash
chmod +x realm
```

## 二、realm配置文件

&emsp;&emsp;realm 的配置文件是 toml 格式的文本文件，用 nano 创建：

```bash
nano /root/realm.toml
```

&emsp;&emsp;其中listen 填本机监听的地址和端口，remote 填目标机器的地址和端口。需要转发多条规则的话，就多写几组 [[endpoints]] ：

```toml
[network]
no_tcp = false
use_udp = true

[[endpoints]]
listen = "0.0.0.0:12345"
remote = "example.com:12345"

[[endpoints]]
listen = "0.0.0.0:54321"
remote = "1.1.1.1:54321"

[[endpoints]]
listen = "[::]:11111"
remote = "[2001:db8::1]:11111"

[[endpoints]]
listen = "0.0.0.0:22222"
remote = "[2001:db8::2]:22222"
```

&emsp;&emsp;其中 listen 里的 [::] 表示监听本机所有 ipv6 地址，0.0.0.0 表示监听所有 ipv4 地址。支持将ipv4转发到ipv6，或反过来。配置好后按 ctrl+x 退出，提示保存时按 y 确认，配置文件就创建好了。

## 三、自启动服务项

&emsp;&emsp;这里创建一个 systemd 服务让 realm 开机自启。也用 nano 创建服务文件：

```bash
nano /etc/systemd/system/realm.service
```

&emsp;&emsp;realm 程序和配置文件都放在 /root 目录下的话，直接写入以下内容即可：

```ini
[Unit]
Description=realm
After=network-online.target
Wants=network-online.target systemd-networkd-wait-online.service

[Service]
Type=simple
User=root
Restart=on-failure
RestartSec=5s
DynamicUser=true
ExecStart=/root/realm -c /root/realm.toml

[Install]
WantedBy=multi-user.target
```

&emsp;&emsp;服务文件创建好后，重载 systemd 配置，然后启动 realm 并设置开机自启：

```bash
systemctl daemon-reload
systemctl enable realm && systemctl start realm
```

&emsp;&emsp;查看 realm 服务的运行状态：

```bash
systemctl status realm
```

&emsp;&emsp;改了配置文件，需要重启服务才能生效：

```bash
systemctl restart realm
```

&emsp;&emsp;如果不再需要转发，可以停止 realm 并取消开机自启：

```bash
systemctl stop realm && systemctl disable realm
```
