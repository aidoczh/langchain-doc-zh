# Fiddler

[Fiddler](https://www.fiddler.ai/) 是企业生成式和预测性系统运营的先驱，提供了一个统一平台，使数据科学、MLOps、风险、合规、分析和其他LOB团队能够监视、解释、分析和改进企业规模的ML部署。

## 1. 安装和设置

```python
#!pip install langchain langchain-community langchain-openai fiddler-client
```

## 2. Fiddler连接细节

*在您可以使用Fiddler添加有关您的模型的信息之前*

1. 您用于连接到Fiddler的URL

2. 您的组织ID

3. 您的授权令牌

这些信息可以在导航到您的Fiddler环境的*设置*页面中找到。

```python
URL = ""  # 您的Fiddler实例URL，请确保包括完整的URL（包括https://）。例如：https://demo.fiddler.ai
ORG_NAME = ""
AUTH_TOKEN = ""  # 您的Fiddler实例授权令牌
# Fiddler项目和模型名称，用于模型注册
PROJECT_NAME = ""
MODEL_NAME = ""  # Fiddler中的模型名称
```

## 3. 创建一个Fiddler回调处理程序实例

```python
from langchain_community.callbacks.fiddler_callback import FiddlerCallbackHandler
fiddler_handler = FiddlerCallbackHandler(
    url=URL,
    org=ORG_NAME,
    project=PROJECT_NAME,
    model=MODEL_NAME,
    api_key=AUTH_TOKEN,
)
```

## 示例1：基本链

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import OpenAI
# 注意：确保在环境变量OPENAI_API_KEY中设置了openai API密钥
llm = OpenAI(temperature=0, streaming=True, callbacks=[fiddler_handler])
output_parser = StrOutputParser()
chain = llm | output_parser
# 调用链。调用将被记录到Fiddler，并自动生成指标
chain.invoke("月球离地球有多远？")
```

```python
# 更多调用
chain.invoke("火星的温度是多少？")
chain.invoke("2 + 200000等于多少？")
chain.invoke("今年奥斯卡奖得主是哪部电影？")
chain.invoke("能写一首关于失眠的诗吗？")
chain.invoke("你今天过得怎么样？")
chain.invoke("生命的意义是什么？")
```

## 示例2：带有提示模板的链

```python
from langchain_core.prompts import (
    ChatPromptTemplate,
    FewShotChatMessagePromptTemplate,
)
examples = [
    {"input": "2+2", "output": "4"},
    {"input": "2+3", "output": "5"},
]
example_prompt = ChatPromptTemplate.from_messages(
    [
        ("human", "{input}"),
        ("ai", "{output}"),
    ]
)
few_shot_prompt = FewShotChatMessagePromptTemplate(
    example_prompt=example_prompt,
    examples=examples,
)
final_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "你是一个奇妙的数学巫师。"),
        few_shot_prompt,
        ("human", "{input}"),
    ]
)
# 注意：确保在环境变量OPENAI_API_KEY中设置了openai API密钥
llm = OpenAI(temperature=0, streaming=True, callbacks=[fiddler_handler])
chain = final_prompt | llm
# 调用链。调用将被记录到Fiddler，并自动生成指标
chain.invoke({"input": "三角形的平方是多少？"})
```