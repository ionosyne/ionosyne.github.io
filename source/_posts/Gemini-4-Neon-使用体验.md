---
title: Gemini 4 Neon 使用体验
date: 2026-11-20 21:50:22
description: 开通 Google AI Pro 后，在 Antigravity CLI 中使用 Gemini 4 Neon 的过程与体验
tags: [AI, Gemini, Antigravity, CLIProxyAPI]
categories: 
- [AI]
---

## 一、写在前面

&emsp;&emsp;最近看到不少人测评Google新发布的gemini-4-neon模型，看起来还不错，刚好最近Google One套餐有折扣，就买了一个试试。当然gemini的网页端一如既往的难用，主要还是试试Antigravity怎么样。

&emsp;&emsp;不过本地使用Antigravity需要一直挂着梯子，很不方便。刚好我有一台服务器上已经部署了 CLIProxyAPI，拿来直接反代 Antigravity成API，这样就可以让本地客户端就不用代理了。当然听说谷歌最近查的很严，会看agent 提示词，所以客户端我还是用的Antigravity CLI，保险一点。


## 二、各种配置

&emsp;&emsp;在 CPA 里面完成 Antigravity 账号登录，就能看到可用的模型了。反代出来的模型基本上都是带思考强度后缀，直接调用会报错，需要做一个模型映射把后缀去掉。

<p align="center">
    <img src="https://img.311305.xyz/eWxEm0DjuL-vuTbIkJMkw3oNF4YUGJWaIBOKyO%2b7iT8%3d/QKK37R72W1G7C.png" style="zoom:50%;" />
</p>


&emsp;&emsp;Antigravity CLI 第三方模型我也不会配置，所以用CC Switch来配置。填好 CPA 的 URL 和 API Key，获取模型列表后选择需要使用的模型，保存即可。需要注意的是，CC Switch的Antigravity 供应商配置不会修改 `agy` 的全局设置。在对应卡片点击 `Open Terminal with This Provider`，可以运行一个单独的终端应用这个配置。直接在终端中运行 `agy`，使用的仍然是它自己的登录和设置。

<p align="center">
    <img src="https://img.311305.xyz/FoiKMS8vlOOX81ZdEMFgURVgVK6BcY0SYAKZypGBwKk%3d/JXUXWV3DDD0ZG.png" style="zoom:30%;" />
</p>


&emsp;&emsp;首次使用Antigravity CLI记得关闭遥测：

<p align="center">
    <img src="https://img.311305.xyz/Fw1eTqN2-LyKsrxGr14q4hT8-QV-FQSEl1GZLe-N09E%3d/UFF8FPF8Q996E.png" style="zoom:50%;" />
</p>

&emsp;&emsp;还有把下面两个域名屏蔽点，防止后台不断上传。我是在hosts里面把这两个域名指向本地来屏蔽的。

``` text
127.0.0.1 paly.googleapis.com
127.0.0.1 daily-cloudcode-pa.googleapis.com
```

## 三、实际体验
&emsp;&emsp;目前实际跑了半天，周限大约用了 30%。比网上别人评测的耐用一点，也可能是token速度不够快。实际用量没有统计，看到别人说gemini-4-neon一周大概有两三亿 token，基本符合体感。而且谷歌最近说要修复缓存问题，后面应该会更耐用一点。再加上 Argon 和 Opus-5.5 的额度，中度使用勉强是够的。

<p align="center">
    <img src="https://img.311305.xyz/mMBuJT%2b-wKim2PTumdNNT6FeOT%2bGjc-gf3DI%2bI2Lyuc%3d/8U4T98RW4C67H.png" style="zoom:50%;" />
</p>


&emsp;&emsp;至于模型本身，体感还不错，个人感觉比 GPT-6.2-Astra强一些，平替Opus-5.5。不会偷懒，说人话，而且前端明显更舒服。不过Antigravity CLI很难用，只能说gemini-4-neon弥补了这一点。总体来讲还是不错的。最近黑五谷歌降价，google one全家桶开一个还是实惠的。
