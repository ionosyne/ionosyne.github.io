---
title: 基于Coze搭建MoA(Mixture-of-Agents)
date: 2024-07-29 20:41:16
description: 利用Coze里的国产大模型结合Coze工作流搭建MoA(Mixture-of-Agents)
tags: [AI, Coze]
categories: 
- [AI]
---

## 一、写在前面

&emsp;&emsp;前段时间看到Together AI发布的MoA([Mixture of Agents](https://arxiv.org/html/2406.04692v1))，通过拼凑很多小模型，在能力上却能和GPT-4一较高下，感觉很有意思。这几天想起来[Coze](https://www.coze.cn/)上刚好有很多国产大模型可以使用，并且Coze自带的工作流模式很适合这种拼凑模型的搭建，就试着捣鼓了一下。

## 二、Mixture-of-Agents

&emsp;&emsp;Mixture-of-Agents的主要思路是通过组合多个大模型来增强能力，多个模型能够减少单一模型可能产生的偏差，使最终答案更加全面和客观。其基本框架如下图所示，主要包括以下几种类型的层：

- **输入层**：负责接收原始输入数据，并将其传递给下一层的多个代理。
- **处理层**：包含多个大型语言模型代理，这些代理独立处理输入，生成初步输出。
- **协作层**：负责汇总和综合来自处理层的结果，并对这些结果进行优化和改进。
- **输出层**：最终的回答或输出在这一层生成，并返回给用户或下游应用。

<p align="center">
    <img src="https://img.311305.xyz/i/2024/09/08/66dd7ec6b2400.png" style="zoom:80%;" />
</p>


&emsp;&emsp;MoA的核心在于处理层和协作层的构建。其中处理层的模型应该具备多样性，能够从不同的角度生成答案，一般会选择能力较弱的模型。协作层的模型需要具备强大的综合和优化能力，能够有效地汇总和分析来自处理层的多个回答。

## 三、Coze MoA工作流搭建

### 1、创建工作流

&emsp;&emsp;进入Coze主页后依次点击 个人空间→工作流→创建工作流 就可以创建一个新的工作流了。新工作流会包含一个开始节点和一个结束节点，对应模型的输入和输出。

<p align="center">
    <img src="https://img.311305.xyz/i/2024/09/08/66dd7f0134095.png"  style="zoom: 80%;" />
</p>


### 2、搭建输入层

&emsp;输入层对应输入节点，这里需要删除多余的变量，如下图所示

<p align="center">
    <img src="https://img.311305.xyz/i/2024/09/08/66dd7f1f21b6a.png" style="zoom: 50%;" />
</p>


### 3、搭建处理层

&emsp;&emsp;这一层的搭建需要点击左边的大模型加号生成新的大模型块，每一块代表一个代理模型。Coze目前可以选择的模型有Baichuan4、GLM-4、MiniMax、Kimi、豆包和通义千问，可以任选几个作为处理层模型。注意这里需要将大模型的块与输入块相连，并定义输入参数。提示词可以直接使用用户输入，当然为了提升模型能力，也可以自己加一些提示词。这里我使用通义千问-8k、MiniMax-8k和Kimi-8k作为代理模型。

<p align="center">
    <img src="https://img.311305.xyz/i/2024/09/08/66dd7f6c76d59.png" style="zoom:50%;" />
</p>


### 4、搭建协作层

&emsp;&emsp;这一层也是采用大模型块，但只有一个模型，这里我选择豆包-fc-32k模型作为协作模型。需要注意的是要把该模型块与上一层模型全部相连，并定义好前面每个模型的输出对应的变量。这里我还额外添加了用户输入作为变量。

<p align="center">
    <img src="https://img.311305.xyz/i/2024/09/08/66dd7f95c6393.png" style="zoom:50%;" />
</p>



&emsp;&emsp;协作层的提示词在文章里提示词的基础上，把原始输入也加了上去：

| You have been provided with a set of responses from various open-source models to the latest user query. Your task is to synthesize these responses into a single, high-quality response. It is crucial to critically evaluate the information provided in these responses, recognizing that some of it may be biased or incorrect. Your response should not simply replicate the given answers but should offer a refined, accurate, and comprehensive reply to the instruction. Ensure your response is well-structured, coherent, and adheres to the highest standards of accuracy and reliability.<br/><br/>Question:<br/>input<br/>Responses from models:<br/>1.input1<br/>2.input2<br/>3.input3 |
| ------------------------------------------------------------ |

### 5、搭建输出层

&emsp;&emsp;输出层对应输出节点，这里将协作层的结果设置作为输出。

<p align="center">
    <img src="https://img.311305.xyz/i/2024/09/08/66dd7fd243925.png" style="zoom:50%;" />
</p>



### 6、整体工作流

&emsp;&emsp;把所有模型块都连起来，整个工作流就完成了。这个工作流可以直接在界面上调试，也可以集成到自己的机器人里用来聊天。

<p align="center">
    <img src="https://img.311305.xyz/i/2024/09/08/66dd800a5c914.png" style="zoom:50%;" />
</p>


&emsp;&emsp;这里简单测试一下。测试问题是：3.11为什么大于3.9？问题比较简单，MoA成功答对了，但整体运行耗时11s。运行速度可能是MoA最大的缺点了吧。

<p align="center">
    <img src="https://img.311305.xyz/i/2024/09/08/66dd8036ee047.png" style="zoom:50%;" />
</p>


## 四、总结

&emsp;&emsp;实际用下来MoA的体验感一般，毕竟一个问题如果所有模型都答不对的话，结果自然也是错的，模型能力并不会有很大提升。不过相比单独的模型，MoA的输出会生动很多。大家感兴趣的话也可以自己去搭建一下MoA，MoA确实是一种可玩性很高的大模型玩法，调一调模型配方和提示词，不同大模型组合也许会碰撞出不一样的火花。
