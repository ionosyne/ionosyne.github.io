---
title: 使用openresty自建DoH
date: 2025-11-08 10:09:46
description: 使用1panel自带的openresty自建DoH服务
tags: [DoH]
categories: 
- [网络]
---

# 一、写在前面

&emsp;&emsp; 目前的公共DoH服务可访问性都比较差，要体验好的话最好是自建，比如说有自己服务器的话可以用nginx反代google、cloudflare的DoH服务达到自建的目的。我比较喜欢使用1panel，docker一把梭的感觉非常好。但是1panel里面用的是openresty，那么用openresty怎么利用反代自建DoH呢。

# 二、openresty配置

&emsp;&emsp;openresty属于nginx的分支，并且有GUI界面，配置起来稍微有点不一样。首先新建一个反向代理，主域名可以直接指向127.0.0.1，这样别人只知道主域名也没法直接使用DoH。

<p align="center">
    <img src="https://img.311305.xyz/UCEGR6GHAW1V.png" style="zoom:33%;" />
</p>

&emsp;&emsp;然后是根据特定前端路径访问DoH，在“配置-反向代理”中可以创建一个新的反向代理指向公告DoH服务，例如google的DoH：https://dns.google/dns-query

<p align="center">
    <img src="https://img.311305.xyz/YTYUTNMHAW1V.png" style="zoom:33%;" />
</p>

&emsp;&emsp;然后编辑该反向代理的源文，修改如下

```nginx
location ^~ /dns-query-google {
    proxy_pass https://dns.google/dns-query;
    proxy_set_header Host dns.google;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

&emsp;&emsp;配置好的效果如下

<p align="center">
    <img src="https://img.311305.xyz/GZRB7YPHAW1V.png" style="zoom:33%;" />
</p>

&emsp;&emsp;因为DoH强制使用https，再配置好SSL证书就可以使用了，通过“域名/前端路径”就可以访问配置好的DoH，也可以设置多个前端路径对应不同的公共DoH。如果是自用的话，可以不进行dns解析，只在hosts中指定ip，这样就不会被别人扫到了。

# 三、如何使用

&emsp;&emsp;现在的浏览器一般都是集成好了DoH功能，例如在chrome浏览器中，可以在“设置-隐私与安全-安全-使用安全DNS”中配置DNS服务商，把自建的DoH域名填进去就可以使用了。

<p align="center">
    <img src="https://img.311305.xyz/E1CWGTUHAW1V.png" style="zoom:33%;" />
</p>

