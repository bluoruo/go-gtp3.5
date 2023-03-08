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
            "content": "使用golang 完成icmp的监听"
        }
    ],
    "model": "gpt-3.5-turbo",
    "max_tokens": 2048,
    "temperature": 0.9,
    "top_p": 1, 
    "stream": false,
    "n":1,
    "presence_penalty": 0,
    "frequency_penalty": 0.6,
    "user": "harry"
}
```
### 说明
#### messages: 
###### 消息主体，和openai交互的信息内容 是数组型
**结构**: \
数组 `json{"role":"","content"}` \
\
**数组内role**: \
system: 是代表给系统一个类型，比如说是开发的，贸易的还是翻译的 \
user: 具体问题内容 \
\
**数组内content**： \
role=system 的时候 代表一个问题的类型 \
role-user 的时候代表问题的具体内容

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
###### 回答内容的最大长度，之所以是token而不是byte，是应为openai返回数据是stream模式，stream交互一次被看做是一个token。所以token<字数
#### temperature: `deafult:1`
###### temper采样率控制,0~2之间。数值越小答案越单一，数值越大答案越随机。<有待进一步调试> 和top_p 互相制约，调试时建议只更改其中一个
#### top_p: `deafult:1`
###### 核心采样率控制,和temperature互相制约。 0.1代表采样率是10%的核心标记
#### n: `deafult:1`
###### 控制放回内容停止在哪里，1代表停止在内容结尾
#### stream: `deafult:false`
###### 是否使用stream模式，stream模式效果类似于ChatGPT网页问答效果，结束标示符 `data: [DONE]`
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
###### 告诉OpenAi消息会话的用户，用来控制违反规则的问话排查

### 弃用的参数
~~suffix: 后缀或者前缀，这个参数已经被最新版本启用~~

## 返回
### Json:
```json
{
    "id": "chatcmpl-abc123123123123",
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
                "content": "以下是使用golang进行icmp监听的示例代码：\n\n```go\npackage main\n\nimport (\n    \"fmt\"\n    \"golang.org/x/net/icmp\"\n    \"golang.org/x/net/ipv4\"\n    \"log\"\n)\n\nfunc main() {\n    conn, err := icmp.ListenPacket(\"ip4:icmp\", \"0.0.0.0\")\n    if err != nil {\n        log.Fatal(err)\n    }\n    defer conn.Close()\n\n    buf := make([]byte, 1024)\n    for {\n        n, addr, err := conn.ReadFrom(buf)\n        if err != nil {\n            log.Fatal(err)\n        }\n        msg, err := icmp.ParseMessage(1, buf[:n])\n        if err != nil {\n            log.Fatal(err)\n        }\n\n        switch msg.Type {\n\t\tcase ipv4.ICMPTypeEcho:\n\t\t\tlog.Printf(\"Received ICMP echo request from %v: %s\\n\", addr.String(), string(msg.Body))\n\t\t\t// 创建ICMP回应消息\n\t\t\techoReplyMsg := icmp.Message{\n\t\t\t\tType: ipv4.ICMPTypeEchoReply,\n\t\t\t\tCode: 0,\n\t\t\t\tBody: &icmp.Echo{\n\t\t\t\t\tID:   msg.Body.(*icmp.Echo).ID,\n\t\t\t\t\tSeq:  msg.Body.(*icmp.Echo).Seq,\n\t\t\t\t\tData: []byte(\"pong\"),\n\t\t\t\t},\n\t\t\t}\n\t\t\techoReplyBytes, _ := echoReplyMsg.Marshal(nil)\n\t\t\tconn.WriteTo(echoReplyBytes, addr)\n\n\t        default:\n\t            log.Printf(\"Received %+v from %v\\n\", msg, addr.String())\n\t        }\n\t    }\n}\n```\n\n运行后，程序将等待来自任何IP地址的ICMP消息，并在接收到ICMP echo请求时发送一个ICMP Echo Reply消息作为回应。"
            },
            "finish_reason": "stop",
            "index": 0
        }
    ]
}
```