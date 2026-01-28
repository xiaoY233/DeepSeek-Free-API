# DeepSeek AI Free 服务

## 项目说明

<span>[ 中文 | <a href="README_EN.md">English</a> ]</span>

支持高速流式输出、支持多轮对话、支持联网搜索、支持R1深度思考和静默深度思考，零配置部署，多路token支持。

本项目由[https://github.com/LLM-Red-Team/deepseek-free-api](https://github.com/LLM-Red-Team/deepseek-free-api)修改而来,感谢大佬的贡献!

提示：当前fork版本未发现恶意代码，欢迎对本项目源码进行审查。

修改原因：

1. 原项目作者账号被封，无法更新了
2. 原项目由于接口更新，无法使用了，根据Fork版本[https://github.com/Fu-Jie/deepseek-free-api](https://github.com/Fu-Jie/deepseek-free-api)进行了调整更新

## 更新说明

1. 模型列表，支持deepseek-chat、deepseek-coder、deepseek-think、deepseek-r1等最新模型

2. 重新打包新版本的docker镜像，`akashrajpuroh1t/deepseek-free-api-fix:latest`

3. 添加了Gemini和Claude适配器，可在gemini-cli和claude-code中调用API

> PS：模型名称实际上并没啥用，只是方便和好看，实际上线上Chat调用是啥模型，就用的啥模型，模型名称随便填都可以。

### 版本说明

- v1.0.0-fix (2025-12-02)
    - 修改默认首页样式，添加接入方式和示例代码
    - 新增Gemini和Claude适配器

## 免责声明

**逆向API是不稳定的，建议前往DeepSeek官方 https://platform.deepseek.com/ 付费使用API，避免封禁的风险。**

**本组织和个人不接受任何资金捐助和交易，此项目是纯粹研究交流学习性质！**

**仅限自用，禁止对外提供服务或商用，避免对官方造成服务压力，否则风险自担！**

**仅限自用，禁止对外提供服务或商用，避免对官方造成服务压力，否则风险自担！**

**仅限自用，禁止对外提供服务或商用，避免对官方造成服务压力，否则风险自担！**

## 效果示例

### 服务默认首页

服务启动后，默认首页添加了接入指南和接口说明，方便快速接入，不用来回切换找文档。

![index.html](./doc/index.png)

### Gemini-cli接入

版本添加了gemini-cli适配器，可以直接在gemini-cli中调用API。

![gemini-cli](./doc/gemini-cli.png)

### Claude-code接入

版本添加了Claude-code适配器，可以直接在Claude-code中调用API。

![claude-code](./doc/claude-code.png)

### 验明正身Demo

![验明正身](./doc/example-1.png)

### 多轮对话Demo

![多轮对话](./doc/example-2.png)

### 联网搜索Demo

![联网搜索](./doc/example-3.png)

## 接入准备

请确保您在中国境内或者拥有中国境内的个人计算设备，否则部署后可能因无法访问DeepSeek而无法使用。

从 [DeepSeek](https://chat.deepseek.com/) 获取userToken value

进入DeepSeek随便发起一个对话，然后F12打开开发者工具，从Application > LocalStorage中找到`userToken`中的value值，这将作为Authorization的Bearer Token值：`Authorization: Bearer TOKEN`

![获取userToken](./doc/example-0.png)

### 多账号接入

目前同个账号同时只能有*一路*输出，你可以通过提供多个账号的userToken value并使用`,`拼接提供：

`Authorization: Bearer TOKEN1,TOKEN2,TOKEN3`

每次请求服务会从中挑选一个。

## Docker部署

请准备一台具有公网IP的服务器并将8000端口开放。

拉取镜像并启动服务

```shell
docker run -it -d --init --name deepseek-free-api -p 8000:8000 -e TZ=Asia/Shanghai akashrajpuroh1t/deepseek-free-api-fix
```

查看服务实时日志

```shell
docker logs -f deepseek-free-api
```

重启服务

```shell
docker restart deepseek-free-api
```

停止服务

```shell
docker stop deepseek-free-api
```

### Docker-compose部署

```yaml
version: '3'

services:
  deepseek-free-api:
    container_name: deepseek-free-api
    image: akashrajpuroh1t/deepseek-free-api-fix:latest
    restart: always
    ports:
      - "8000:8000"
    environment:
      - TZ=Asia/Shanghai
```

## 接口列表

目前支持：

1. 与OpenAI兼容的 `/v1/chat/completions` 接口
2. 与Google Gemini兼容的 `/v1beta/models/:model:generateContent` 接口  
3. 与Anthropic Claude兼容的 `/v1/messages` 接口

可自行使用与openai、gemini-cli、claude-code或其他兼容的客户端接入接口，或者使用 [dify](https://dify.ai/) 等线上服务接入使用。

### 对话补全

对话补全接口，与openai的 [chat-completions-api](https://platform.openai.com/docs/guides/text-generation/chat-completions-api) 兼容。

**POST /v1/chat/completions**

header 需要设置 Authorization 头部：

```
Authorization: Bearer [userToken value]
```

请求数据：
```json
{
    // model名称
    // 默认：deepseek
    // 深度思考：deepseek-think 或 deepseek-r1
    // 联网搜索：deepseek-search
    // 深度思考+联网搜索：deepseek-r1-search 或 deepseek-think-search
    // 静默模式（不输出思考过程或联网搜索结果）：deepseek-think-silent 或 deepseek-r1-silent 或 deepseek-search-silent
    // 深度思考但思考过程使用<details>可折叠标签包裹（需要页面支持显示）：deepseek-think-fold 或 deepseek-r1-fold
    "model": "deepseek",
    // 默认多轮对话基于消息合并实现，某些场景可能导致能力下降且受单轮最大token数限制
    // 如果您想获得原生的多轮对话体验，可以传入上一轮消息获得的id，来接续上下文
    // "conversation_id": "50207e56-747e-4800-9068-c6fd618374ee@2",
    "messages": [
        {
            "role": "user",
            "content": "你是谁？"
        }
    ],
    // 如果使用流式响应请设置为true，默认false
    "stream": false
}
```

响应数据：
```json
{
    "id": "50207e56-747e-4800-9068-c6fd618374ee@2",
    "model": "deepseek",
    "object": "chat.completion",
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": " 我是DeepSeek Chat，一个由深度求索公司开发的智能助手，旨在通过自然语言处理和机器学习技术来提供信息查询、对话交流和解答问题等服务。"
            },
            "finish_reason": "stop"
        }
    ],
    "usage": {
        "prompt_tokens": 1,
        "completion_tokens": 1,
        "total_tokens": 2
    },
    "created": 1715061432
}
```

### userToken存活检测

检测userToken是否存活，如果存活live为true，否则为false，请不要频繁（小于10分钟）调用此接口。

**POST /token/check**

请求数据：
```json
{
    "token": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9..."
}
```

响应数据：
```json
{
    "live": true
}
```
