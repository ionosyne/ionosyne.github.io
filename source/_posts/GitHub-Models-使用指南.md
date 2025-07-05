---
title: GitHub Models 使用指南
date: 2024-08-10 10:33:26
description: 把GitHub Models接入自己的AI应用
tags: [AI, GitHub]
categories: 
- [AI]
---

## 一、GitHub Models

&emsp;&emsp;[GitHub Models](https://github.com/marketplace/models) 是GitHub最新推出的模型托管服务，提供免费的AI模型供开发者测试。可以使用的模型有 GPT-4o、Phi 3、Llama 3.1等，可以说很全面的了。现在可以申请加入[waitlist](https://github.com/marketplace/models/waitlist)，通过后就可以使用这些模型了。

## 二、模型调用

&emsp;&emsp;GitHub Models提供了Playground 进行调试，当然也可以直接使用API。例如使用cURL请求:

```bash
curl -X POST "https://models.inference.ai.azure.com/chat/completions" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $GITHUB_TOKEN" \
    -d '{
        "messages": [
            {
                "role": "system",
                "content": "You are a helpful assistant."
            },
            {
                "role": "user",
                "content": "What is the capital of France?"
            }
        ],
        "model": "gpt-4o"
    }'
```

&emsp;&emsp;调用方式和OpenAI基本一致，只是请求地址从api.openai.com/v1变成了models.inference.ai.azure.com，使用[GITHUB_TOKEN](https://github.com/settings/tokens)作为apikey。

## 三、接入 one-api

&emsp;&emsp;我喜欢用one-api来进行api管理。但是GitHub Models的请求地址因为少了 `/v1` 没办法直接接入one-api。这里选择用cloudflare workers做一个接口转发，去除请求地址中的`/v1`：

```javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)

  if (url.pathname.startsWith('/v1')) {
    const newUrl = new URL(request.url)
    newUrl.hostname = 'models.inference.ai.azure.com'
    newUrl.pathname = newUrl.pathname.replace('/v1', '') 

    const newRequest = new Request(newUrl, {
      method: request.method,
      headers: request.headers,
      body: request.body
    })

    return fetch(newRequest)
  }

  return fetch(request)
}
```

&emsp;&emsp;使用worker的URL作为API地址，GITHUB_TOKEN作为apikey就可以将GitHub Models接入one-api了。

## 四、使用体验

&emsp;&emsp;因为已经集成到one-api了，所以可以很方便的在各种AI应用中调试，例如在open-webui里使用：

<p align="center">
    <img src="https://p.iz.mk/i/2025/07/05/6868ba421a534.webp" style="zoom:80%;" />
</p>

&emsp;&emsp;响应速度还是挺快的，不过和官方模型用起来有些差异，应该是版本有些不同。最后是模型使用限制，GPT-4o免费用户上下文限制到了8k，每分钟请求最大为10，每天最多请求50次。用来测试应用的话应该是够用的。
