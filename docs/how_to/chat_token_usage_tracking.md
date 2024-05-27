

# 如何跟踪 ChatModels 中的令牌使用情况

:::info 先决条件

本指南假定您熟悉以下概念：

- [Chat 模型](/docs/concepts/#chat-models)

:::

跟踪令牌使用情况以计算成本是将您的应用投入生产的重要部分。本指南介绍了如何从您的 LangChain 模型调用中获取此信息。

## 使用 AIMessage.response_metadata

许多模型提供程序将令牌使用信息作为聊天生成响应的一部分返回。如果可用，这将包含在 [`AIMessage.response_metadata`](/docs/how_to/response_metadata) 字段中。以下是一个使用 OpenAI 的示例：

```python
# !pip install -qU langchain-openai
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-4-turbo")
msg = llm.invoke([("human", "最古老的楔形文字的已知例子是什么")])
msg.response_metadata
```

```output
{'token_usage': {'completion_tokens': 225,
  'prompt_tokens': 17,
  'total_tokens': 242},
 'model_name': 'gpt-4-turbo',
 'system_fingerprint': 'fp_76f018034d',
 'finish_reason': 'stop',
 'logprobs': None}
```

这里是一个使用 Anthropic 的示例：

```python
# !pip install -qU langchain-anthropic
from langchain_anthropic import ChatAnthropic
llm = ChatAnthropic(model="claude-3-sonnet-20240229")
msg = llm.invoke([("human", "最古老的楔形文字的已知例子是什么")])
msg.response_metadata
```

```output
{'id': 'msg_01P61rdHbapEo6h3fjpfpCQT',
 'model': 'claude-3-sonnet-20240229',
 'stop_reason': 'end_turn',
 'stop_sequence': None,
 'usage': {'input_tokens': 17, 'output_tokens': 306}}
```

## 使用回调

还有一些特定于 API 的回调上下文管理器，允许您跟踪多个调用中的令牌使用情况。目前仅为 OpenAI API 和 Bedrock Anthropic API 实现了此功能。

### OpenAI

让我们首先看一个极其简单的示例，用于跟踪单个 Chat 模型调用的令牌使用情况。

```python
# !pip install -qU langchain-community wikipedia
from langchain_community.callbacks.manager import get_openai_callback
llm = ChatOpenAI(model="gpt-4-turbo", temperature=0)
with get_openai_callback() as cb:
    result = llm.invoke("告诉我一个笑话")
    print(cb)
```

```output
使用的令牌数：26
	提示令牌：11
	完成令牌：15
成功请求次数：1
总成本（美元）：$0.00056
```

上下文管理器中的任何内容都将被跟踪。以下是在其中使用它来跟踪连续多次调用的示例。

```python
with get_openai_callback() as cb:
    result = llm.invoke("告诉我一个笑话")
    result2 = llm.invoke("告诉我一个笑话")
    print(cb.total_tokens)
```

```output
52
```

如果使用具有多个步骤的链或代理，它将跟踪所有这些步骤。

```python
from langchain.agents import AgentExecutor, create_tool_calling_agent, load_tools
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "您是一个乐于助人的助手"),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ]
)
tools = load_tools(["wikipedia"])
agent = create_tool_calling_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent, tools=tools, verbose=True, stream_runnable=False
)
```

:::note

我们必须将 `stream_runnable=False` 设置为令牌计数才能正常工作。默认情况下，AgentExecutor 将流式传输底层代理，以便在通过 AgentExecutor.stream_events 流式传输事件时获得最精细的结果。但是，OpenAI 在流式传输模型响应时不会返回令牌计数，因此我们需要关闭底层流式传输。

:::

```python
with get_openai_callback() as cb:
    response = agent_executor.invoke(
        {
            "input": "蜂鸟的学名是什么，哪种鸟是最快的？"
        }
    )
    print(f"总令牌数：{cb.total_tokens}")
    print(f"提示令牌：{cb.prompt_tokens}")
    print(f"完成令牌：{cb.completion_tokens}")
    print(f"总成本（美元）：${cb.total_cost}")
```

```output
> 进入新的 AgentExecutor 链...
调用：使用 `wikipedia` 查找 `蜂鸟`
页面：蜂鸟
摘要：蜂鸟是原产于美洲的鸟类，包括生物学家 Trochilidae 家族。约有 366 种和 113 个属，它们分布从阿拉斯加到火地岛，但大多数物种分布在中美洲和南美洲。截至 2024 年，有 21 种蜂鸟被列为濒危或极度濒危物种，许多物种的数量在下降。蜂鸟具有各种专门化特征，以实现快速、灵活的飞行：卓越的代谢能力、适应高海拔、敏锐的视觉和沟通能力，以及一些物种的远距离迁徙。在所有鸟类中，雄性蜂鸟的羽毛颜色最为丰富，尤其是蓝色、绿色和紫色。蜂鸟是最小的成年鸟类，体长为 7.5–13 厘米（3–5 英寸）。最小的是 5 厘米（2.0 英寸）的蜂鸟，体重不到 2.0 克（0.07 盎司），最大的是 23 厘米（9 英寸）的巨型蜂鸟，体重为 18–24 克（0.63–0.85 盎司）。蜂鸟以长喙而闻名，专门用于吸食花蜜，但所有物种也吃小昆虫。
它们因翅膀拍打时发出的嗡嗡声而被称为蜂鸟，这种声音频率高，其他鸟类和人类都能听到。它们以快速的翅膀拍打速率盘旋，从最大物种的每秒约 12 次拍打到小蜂鸟的每秒 80 次。
蜂鸟具有任何恒温动物中最高的质量特异代谢率。为了在食物稀缺和夜晚不觅食时节省能量，它们可以进入类似冬眠的状态，将其代谢率减慢到正常速率的 1/15。虽然大多数蜂鸟不迁徙，但红胁蜂鸟是鸟类中迁徙距离最长的之一，每年在阿拉斯加和墨西哥之间往返两次，距离约为 3,900 英里（6,300 公里）。
蜂鸟大约在 4200 万年前与其姐妹群体，雨燕和树雨燕分开。已知最古老的化石蜂鸟是来自早渐新世欧洲的 Eurotrochilus。
页面：蜂鸟蜂鸟
摘要：蜂鸟蜂鸟，zunzuncito 或 Helena 蜂鸟（Mellisuga helenae）是一种蜂鸟，原产于加勒比海古巴岛。它是已知最小的鸟类。蜂鸟蜂鸟以古巴的花朵和虫子的花蜜为食。
页面：蜂鸟蛋糕
摘要：蜂鸟蛋糕是一种起源于牙买加的香蕉菠萝香料蛋糕，自 1970 年代以来在美国南部地区很受欢迎。成分包括面粉、糖、盐、植物油、熟香蕉、菠萝、肉桂、山核桃、香草提取物、鸡蛋和发酵剂。通常与奶油奶酪糖霜一起食用。
调用：使用 `wikipedia` 查找 `最快的鸟`
页面：最快的动物
摘要：这是世界上最快动物的列表，按动物类型分类。
页面：按飞行速度列出的鸟类
摘要：这是世界上飞行速度最快的鸟类列表。鸟类的速度必然是可变的；捕猎鸟在俯冲捕捉猎物时将达到更高的速度，而在水平飞行时速度较低。能够达到最大空速的鸟类是隼，能够在俯冲时超过 320 公里/小时（200 英里/小时）。普通雨燕的近亲白喉针尾雨燕（Hirundapus caudacutus）通常被报道为水平飞行中速度最快的鸟类，据报道最高速度为 169 公里/小时（105 英里/小时）。这一记录尚未得到确认，因为测量方法从未被公布或验证。鸟类经过确认的水平飞行速度记录为 111.5 公里/小时（69.3 英里/小时），由普通雨燕保持。
页面：鸵鸟
摘要：鸵鸟是大型不会飞行的鸟类。它们是最重和最大的现存鸟类，成年普通鸵鸟的体重在 63.5 至 145 千克之间，产下的蛋是所有陆地动物中最大的。它们能够以每小时 70 公里的速度奔跑，是陆地上最快的鸟类。它们在全球范围内被饲养，菲律宾和纳米比亚有重要的产业。鸵鸟皮革是一种有利可图的商品，大羽毛被用作装饰仪式头饰的羽毛。鸵鸟蛋已被人类使用数千年。
鸵鸟属于鸵形目的鸵鸟属，属于下纲 Palaeognathae 的一部分，这是一个包括飞行鸟类、雷鸟、几种鸵鸟、几种几维鸟和灭绝的大象鸟和恐鸟在内的多样化的不会飞行的鸟类群。鸵鸟有两种现存物种：普通鸵鸟，原产于撒哈拉以南非洲的大片地区，和索马里鸵鸟，原产于非洲之角。普通鸵鸟历史上曾是阿拉伯半岛的本地物种，且在晚更新世期间至少到中国和蒙古。### 蜂鸟的学名
蜂鸟的学名是 **Mellisuga helenae**，它是已知最小的鸟类，也是一种蜂鸟。
### 最快的鸟类
就空速而言，最快的鸟是 **隼**，在俯冲飞行时速度可超过 320 公里/小时（200 英里/小时）。在水平飞行中，最快的确认速度由 **普通雨燕** 保持，它可以以 111.5 公里/小时（69.3 英里/小时）的速度飞行。
> 链结束。
总令牌数：1583
提示令牌：1412
完成令牌：171
总成本（美元）：$0.019250000000000003
```

### Bedrock Anthropic

`get_bedrock_anthropic_callback` 的工作原理非常相似：

```python
# !pip install langchain-aws
from langchain_aws import ChatBedrock
from langchain_community.callbacks.manager import get_bedrock_anthropic_callback
llm = ChatBedrock(model_id="anthropic.claude-v2")
with get_bedrock_anthropic_callback() as cb:
    result = llm.invoke("Tell me a joke")
    result2 = llm.invoke("Tell me a joke")
    print(cb)
```

```output
Tokens Used: 0
    Prompt Tokens: 0
    Completion Tokens: 0
Successful Requests: 2
Total Cost (USD): $0.0
```

## 下一步

您已经看到了如何跟踪支持提供商的令牌使用的一些示例。

接下来，查看本节中其他如何指南中的聊天模型，比如[如何让模型返回结构化输出](/docs/how_to/structured_output)或[如何为您的聊天模型添加缓存](/docs/how_to/chat_model_caching)。