---
title: 使用BabelDOC一键翻译pdf
date: 2025-08-03 13:57:15
description: 通过简单配置babeldoc，使用AI一键翻译pdf
tags: [AI, translate]
categories: 
- [软件]
---

## 一、写在前面
&emsp;&emsp;最近买了智谱GLM-4.5 50块包月的资源包，token太多没地方用也是一种焦虑。正好平时读文章比较多，就想如果能用GLM模型一键翻译论文，那么阅读文章的效率就会大大提升，这些token也算派上用场。

&emsp;&emsp;这里推荐BabelDOC作为翻译程序，可以接入AI模型，主要是公式排版非常好：

<div class="custom-link-card" style="border: 1px solid #e1e4e8; border-radius: 6px; margin: 1.5em 0; padding: 16px; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji';">
  <a href="https://github.com/funstory-ai/BabelDOC" target="_blank" style="text-decoration: none;">
    <h3 style="font-size: 20px; font-weight: 400; color: #1a0dab; margin: 0 0 3px; padding: 0;">
      funstory-ai/BabelDOC - GitHub
    </h3>
  </a>
  <div class="cite" style="font-size: 14px; color: #006621; font-style: normal; margin-bottom: 8px;">
    https://github.com/funstory-ai/BabelDOC
  </div>
  <div class="description" style="font-size: 14px; color: #586069; line-height: 1.5;">
    AI-powered multi-language document translation tool that maintains the original format. It supports various formats such as PDF, Word, Excel, PowerPoint, and Markdown.
  </div>
</div>

&emsp;&emsp;BabelDOC现在可以在沉浸式翻译上面使用，zotero的pdf2zh插件也集成了，但是用起来还是不方便。这里就介绍一种可以通过简单配置BabelDOC一键翻译pdf的方法。

## 二、BabelDOC安装

&emsp;&emsp;BabelDOC安装推荐用uv进行，需要电脑上装有python并且安装了uv包，使用uv安装BabelDOC的命令如下：

```bash
uv tool install --python 3.12 BabelDOC

babeldoc --help
```

&emsp;&emsp;如果出现下图说明安装成功，如果报错提示babeldoc不是系统命令，可以尝试将`C:\Users\username\Desktop\.local\bin`加入环境变量再运行`babeldoc --help`试试

<p align="center">
    <img src="https://p.iz.mk/i/2025/08/03/688f044c46621.webp" style="zoom:75%;" />
</p>

## 三、BabelDOC配置

&emsp;&emsp;我们要通过命令行的方式调用babeldoc，新建一个如下的`.bat`文件，这里的openai模型配置需要替换成自己的模型和api-key：

```bash
@echo off
set "filepath=%~1"
set "filedir=%~dp1"
pushd "%filedir%"

babeldoc --openai --openai-model "glm-4.5" --openai-base-url "https://open.bigmodel.cn/api/paas/v4" --openai-api-key "your-api-key" --files "%filepath%"
```

&emsp;&emsp;最后为这个`.bat`文件生成一个**快捷方式**，名称和图标可以自己选，然后把快捷方式放到`%APPDATA%\Microsoft\Windows\SendTo`目录下。放到这个目录下的文件会出现在右键菜单-发送到。我们右键pdf发送到babeldoc，就可以使用上面的bat处理pdf了：

<p align="center">
    <img src="https://p.iz.mk/i/2025/08/03/688f06899e10e.webp" style="zoom:33%;" />
</p>

## 四、使用测试

&emsp;&emsp;这里用一篇`Attention Is All You Need`来进行测试。如果是第一次使用的话babeldoc会下载必要的字体，速度会比较慢。过程中warning也挺多的，直接无视就好。因为我的GLM-4.5并发有点低，而且默认是思考模型，ai翻译花了8分钟，还有4分钟用来处理文档，总共花了12分钟和50W token。

<p align="center">
    <img src="https://p.iz.mk/i/2025/08/03/688f07c50857c.webp" style="zoom:75%;" />
</p>

&emsp;&emsp;翻译成功后会在原目录生成两个文件，一个是英文翻译pdf，一个是中英对照pdf。GLM在翻译这块确实可以，配合上BabelDOC的公式和格式，整体效果可以说赏心悦目：

<p align="center">
    <img src="https://p.iz.mk/i/2025/08/03/688f08ac580e3.webp" style="zoom:75%;" />
</p>

&emsp;&emsp;当然翻译速度还是和机器配置以及api速度挂钩，如果GLM的并发和吐字速度能再高一点的话，整体体验应该会更好。大家感兴趣的话也可以自己去试试配置配置，看看效果如何。