---
title: 云端自动化编译Docker镜像为Singularity镜像
date: 2025-05-17 20:13:06
description: 一个完整的自动化工作流，用于将Docker镜像编译为Singularity/Apptainer(.sif)镜像
tags: [docker, singularity, workflow]
categories: 
- [软件]
---

## 一、开篇

&emsp;&emsp;最近需要在服务器上安装环境，我最先想到的就是使用已有的docker环境，简单又方便。不过尴尬的是服务器上只能使用singularity。难道要重新折腾一遍，我本地也没装singularity啊？好像singularity也是支持docker镜像的吧？把docker镜像编译为singularity好像也还行，那就研究一下吧。

## 二、GitHub Actions工作流搭建

&emsp;&emsp;既然本地没有安装singularity，那还是折腾云端会比较好，就比如常用的GitHub Actions。说干就干，先写一个`build-singularity.yml`的workflow文件：

```yml
name: Build Singularity Image from Docker

on:
  workflow_dispatch:

jobs:
  build-singularity:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      actions: write  

    steps:
      - name: Install Apptainer (Singularity)
        run: |
          sudo apt-get update
          sudo apt-get install -y software-properties-common  
          sudo add-apt-repository universe -y  
          sudo apt-get update  
          sudo apt-get install -y wget uuid-dev libseccomp-dev pkg-config squashfs-tools cryptsetup libfuse2 squashfuse fuse2fs
          export VER=1.1.9
          wget https://github.com/apptainer/apptainer/releases/download/v${VER}/apptainer_${VER}_amd64.deb
          sudo dpkg -i apptainer_${VER}_amd64.deb
      
      - name: Build Singularity Image from Docker Image
        run: |
          apptainer build my_image.sif docker://alpine:latest
      
      - name: Upload SIF as Artifact
        uses: actions/upload-artifact@v4  
        with:
          name: sif-image
          path: my_image.sif
```

&emsp;&emsp;这里比较坑的一点是现在singularity改名成apptainer了，所以需要安装apptainer而不是singularity。这里修改`apptainer build my_image.sif docker://my_image:latest`中镜像的名称来编译自己的docker镜像。

&emsp;&emsp;完成上面的准备工作后，新建一个github仓库，把这个文件上传到`/.github/workflows`文件夹下面，github action就会自动开始编译工作了。测试的镜像编译的很顺利，编译完成后的SIF文件会上传到Artifact，可以通过链接下载。

<p align="center">
    <img src="https://p.iz.mk/i/2025/07/07/686bc2b1192c2.webp" style="zoom:75%;" />
</p>

&emsp;&emsp;不过不出意外的话，马上就要出意外了。当我准备编译自己的镜像的时候，发现虚拟环境的空间竟然不够了。额，我的镜像大概10个G，确实大了一点。

<p align="center">
    <img src="https://p.iz.mk/i/2025/07/07/686bc5129c9b3.webp" style="zoom:75%;" />
</p>

&emsp;&emsp;那在安装环境之前先清一下空间怎么样，比如这样：

```yml
    steps:
      - name: Free Disk Space (Ubuntu)
        uses: jlumbroso/free-disk-space@main
        with:
          tool-cache: true
    
          android: true
          dotnet: true
          haskell: true
          large-packages: true
          swap-storage: true
```

&emsp;&emsp;这个命令大概可以清理出50G左右的空间：

<p align="center">
    <img src="https://p.iz.mk/i/2025/07/07/686bc67a84f79.webp" style="zoom:50%;" />
</p>

&emsp;&emsp;不过还是不行，最后还是报空间不足，看来将docker镜像编译为singularity空间要求太大了，github action只适合小镜像的编译。

## 三、Google Colab云端编译

&emsp;&emsp;那有没有空间更大一点的虚拟环境呢？那当然是有的，比如Google Colab提供了大概108G的空间，非常充足。而且Google Colab不仅可以执行python命令，也是可以执行bash指令的（命令前面加！）。

&emsp;&emsp;那就开干，首先安装apptainer：

```bash
!sudo add-apt-repository -y ppa:apptainer/ppa
!sudo apt update
!sudo apt install -y apptainer
```

&emsp;&emsp;测试一下编译命令，perfect！！

<p align="center">
    <img src="https://p.iz.mk/i/2025/07/07/686bc90228b3e.webp" style="zoom:75%;" />
</p>

&emsp;&emsp;然后就是用命令`!apptainer build my_image.sif docker://my_image:latest`编译自己的镜像了，这回总算编译成功了，峰值的时候用掉了大概80G左右的空间，差点又编译失败了。不过Google Colab速度比GitHub Actions慢了不止一点，如果镜像不是很大的话还是优选考虑GitHub Actions。

<p align="center">
    <img src="https://p.iz.mk/i/2025/07/07/686bc9cc9b0d9.webp" style="zoom:50%;" />
</p>

&emsp;&emsp;编译完成后的SIF文件会位于colab的文件目录里，直接下的话会很卡，建议挂载自己的google drive然后拷进去。从google drive里面下会顺畅很多。

## 四、总结

&emsp;&emsp;好了现在有了SIF文件，可以上传到服务器尽情折腾了，虽然这一步好像更磨人。
