# GTP3.5接口

## 提交
**URL**:\
`https://api.openai.com/v1/chat/completions` \
**Http_Method**: \
`POST` \
**Http_Header**: \
`Authorization : Bearer+[空格]+$Api_Key` \
`OpenAI-Organization : $Organization_ID`
### Json :
```json
{
    "messages": [
        {
            "role": "system",
            "content": "developer"
        },
        {
            "role": "user",
            "content": "你好"
        }, 
        {
            "role": "assistant",
            "content": "好"
        },
        {
            "role": "user",
            "content": "使用golang 完成HelloWorld"
        }
    ],
    "model": "gpt-3.5-turbo",
    "max_tokens": 2048,
    "temperature": 0.7,
    "top_p": 1, 
    "stream": false,
    "n":1,
    "presence_penalty": 0.6,
    "frequency_penalty": 0.6,
    "user": "bluoruo"
}
```
### 请求Json说明
#### messages: 
###### 消息主体，程序和openai交互的信息内容 是数组类型

**结构**: \
数组 `json{"role":"","content":""}` 

**数组内role**: \
system: 是代表给openai的一个参考，比如说是开发的、新闻、文字编辑或者语言翻译等 \
user: 具体问题内容 \
新版本在此处可以加入联系上下文的功能，把上次对话的整个内容保存在messages消息内容内。\
GPT-3.5会更好的处理您的相关问题，而不是简单回答当前问题。

**数组内content**： \
role=system 的时候代表一个话题的类型\
role=user 的时候代表话题的具体内容

#### model:
###### 使用那种模型作为引擎，GTP-3.5推荐模型列表：
| model              | 说明                | 最大请求 |   训练数据 |
|:-------------------|:------------------|:----:|-------:|
| gpt-3.5-turbo      | 聊天优化,模型迭代         | 4096 | 2021-9 |
| gpt-3.5-turbo-0301 | 聊天优化,模型只在2023-3-1 | 4096 | 2021-9 |
| text-davinci-003   | 文本处理和翻译优化，支持文本补全  | 4000 | 2021-9 |
| text-davinci-002   | 类似003但训练有素        | 4000 | 2021-9 |
| code-davinci-002   | 代码处理进行优化          | 4000 | 2021-9 |

#### max_tokens: `deafult:inf`
###### 回答内容的最大长度，之所以是tokens而不是bytes，个人理解：是因为openai返回数据是stream模式，stream交互一次被看做是一个token
#### temperature: `deafult:1`
###### 采样率控制,0~2之间。数值越小回答越单一，数值越大答案越随机。<有待进一步调试> temperature和top_p 互相制约，建议只更改其中一个
#### top_p: `deafult:1`
###### 核心采样率控制,top_p和temperature互相制约。 0.1代表采样率是10%的核心标记
#### n: `deafult:1`
###### 控制返回内容停止在哪里，1代表停止在内容结尾
#### stream: `deafult:false`
###### 是否使用stream模式，stream模式效果类似于ChatGPT网页问答效果。结束符 `data: [DONE]`
#### stop: `deafult:null`
###### 立即停止，告诉openai停止生成更多的tokens
#### presence_penalty:
###### 新话题规则。-2.0~2.0之间，大于零的数值，会根据到目前为止是否出现在文本中来判断新标记，从而增加模型谈论新主题的可能性
#### frequency_penalty:
###### 防重复规则。-2.0~2.0之间，大于零的数值，会根据新标记在文本中的现有频率对其进行判断，从而降低模型逐字重复同一行的可能性
#### logit_bias:
###### 修改指定标记出现在完成中的可能性
(接受一个 json 对象，该对象将标记（由标记器中的标记 ID 指定）映射到从 -100 到 100 的关联偏差值。\
从数学上讲，偏差会在采样之前添加到模型生成的 logits 中。\
确切的效果因模型而异，但 -1 和 1 之间的值应该会减少或增加选择的可能性；\
像 -100 或 100 这样的值应该导致相关令牌的禁止或独占选择。)
#### user:
###### 告诉OpenAi消息会话的用户，用来控制违反规则的会话

### 弃用的参数
~~suffix: 后缀或者前缀，这个参数已经被最新版本启用~~

## 返回
### Json:
```json
{
    "id": "chatcmpl-abc123123123123xxx",
    "object": "chat.completion",
    "created": 1678252628,
    "model": "gpt-3.5-turbo-0301",
    "usage": {
        "prompt_tokens": 22,
        "completion_tokens": 378,
        "total_tokens": 400
    },
    "choices": [
        {
            "message": {
                "role": "assistant",
                "content": "下面是一个简单的Go程序，输出\"Hello World!\"。\n\n```go\npackage main\n\nimport \"fmt\"\n\nfunc main() {\n    fmt.Println(\"Hello World!\")\n}\n```\n\n打开命令行工具，执行以下命令运行程序：\n\n```\ngo run hello.go\n```\n\n这将在终端中打印出 \"Hello World!\"。"
            },
            "finish_reason": "stop",
            "index": 0
        }
    ]
}
```
### 返回Json说明
#### id: `chatcmpl-xxxxxx`
###### openai给出的会话ID,可用于调试或者排查。
#### object: `chat.completion`
###### 使用的引擎
#### created:
###### 会话创建的时间，unix时间格式
#### model:
###### 使用的引擎
#### usage:
###### 使用的tokens信息，可以用来监控和控制使用openai的成本。
**prompt_tokens**: 用户请求的tokens数量\
**completion_tokens**: openai处理的tokens数量\
**total_tokens**: 本次会话总共tokens数量
#### choices: 
###### 会话返回的主消息体
**message**: 消息体内容\
message.role: 消息类型 \
message.content: 消息内容 

**finish_reason**:  \
stop: 已经完整的返回了内容 \
length: 因为max_tokens参数，限制了输出内容 \
content_filter: 内过滤了 \
null: api响应中，或者处理出错了 

**index**: 索引，暂时不理解