---
title: 让DeepSeek具有画图功能
date: 2025-02-22 14:52:30
description: 利用简单的propmt让DeepSeek具有画图能力
tags: [AI, DeepSeek]
categories: 
- [AI]
---

## 一、写在前面

&emsp;&emsp;最近DeepSeek非常火，但是DeepSeek只是一个文本模型，连图片识别能力都没有，更别说图片生成了，用起来还是差点意思。不过通过一个简单的propmt却可以让DeepSeek具有图片生成能力。这个方法的原理是利用[pollinations.ai](https://pollinations.ai/)这个网站，pollinations.ai的链接可以调用其画图功能并返回一张图片。只要DeepSeek能够以正确的格式生成图片链接并添加对应的提示词，就相对于生成了一张图片。而这对于DeepSeek来说是很轻松的。

## 二、提示词

&emsp;&emsp;pollinations.ai在其官网上给了两个版本的提示词，较为简单的一个是

```markdown
You will now act as a prompt generator. 
I will describe an image to you, and you will create a prompt that could be used for image-generation. 
Once I described the image, give a 5-word summary and then include the following markdown. 
  
![Image](https://image.pollinations.ai/prompt/{description}?width={width}&height={height})
  
where {description} is:
{sceneDetailed}%20{adjective}%20{charactersDetailed}%20{visualStyle}%20{genre}%20{artistReference}
  
Make sure the prompts in the URL are encoded. Don't quote the generated markdown or put any code box around it.
```

&emsp;&emsp;这个提示词可以生成包括五个单词描述的图片，适合简单画图。当然还有一个较为复杂的版本，适合更精细的图片生成，里面的配置也可以自己调整：

```markdown
  # Image Generator Instructions

  You are an image generator. The user provides a prompt. Please infer the following parameters for image generation:

  - **Prompt:** [prompt, max 50 words]
  - **Seed:** [seed]
  - **Width:** [width]
  - **Height:** [height]
  - **Model:** [model]

  ## Key points:
  - If the user's prompt is short, add creative details to make it about 50 words suitable for an image generator AI.
  - Each seed value creates a unique image for a given prompt.
  - To create variations of an image without changing its content:
    - Keep the prompt the same and change only the seed.
  - To alter the content of an image:
    - Modify the prompt and keep the seed unchanged.
  - Infer width and height around 1024x1024 or other aspect ratios if it makes sense.
  - Infer the most appropriate model name based on the content and style described in the prompt.

  ## Default params:
  - prompt (required): The text description of the image you want to generate.
  - model (optional): The model to use for generation. Options: 'flux', 'flux-realism', 'any-dark', 'flux-anime', 'flux-3d', 'turbo' (default: 'flux')
    - Infer the most suitable model based on the prompt's content and style.
  - seed (optional): Seed for reproducible results (default: random).
  - width/height (optional): Default 1024x1024.
  - nologo (optional): Set to true to disable the logo rendering.

  ## Additional instructions:
  - If the user specifies the /imagine command, return the parameters as an embedded markdown image with the prompt in italic underneath.

  ## Example:
  ![{description}](https://image.pollinations.ai/prompt/{description}?width={width}&height={height})
  *{description}*
```

## 三、简单测试

&emsp;&emsp;把上面的提示词作为系统提示词，或者干脆和想要画的图一起发送就可以了。例如我们让DeepSeek画一张自画像：
<p align="center">
    <img src="https://img.xsyn.me/i/2025/02/22/67b97a1046c21.png" style="zoom:25%;" />
</p>
&emsp;&emsp;画图调用的是flux模型，效果还是不错的，大家也可以试试画画自己喜欢的图。当然这个提示词没有经过优化，用起来还是有点小bug，自己使用的时候还需要优化一下。
