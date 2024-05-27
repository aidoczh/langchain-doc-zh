# Cloudflare Workers AI
[Cloudflare AI文档](https://developers.cloudflare.com/workers-ai/models/text-generation/)列出了所有可用的生成文本模型。
需要Cloudflare账户ID和API令牌。从[此文档](https://developers.cloudflare.com/workers-ai/get-started/rest-api/)中找到如何获取它们。
```python
from langchain.chains import LLMChain
from langchain_community.llms.cloudflare_workersai import CloudflareWorkersAI
from langchain_core.prompts import PromptTemplate
template = """Human: {question}
AI Assistant: """
prompt = PromptTemplate.from_template(template)
```
在运行LLM之前进行身份验证。
```python
import getpass
my_account_id = getpass.getpass("输入您的Cloudflare账户ID：\n\n")
my_api_token = getpass.getpass("输入您的Cloudflare API令牌：\n\n")
llm = CloudflareWorkersAI(account_id=my_account_id, api_token=my_api_token)
```
```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "为什么玫瑰是红色的？"
llm_chain.run(question)
```
```output
"AI Assistant: 啊，一个迷人的问题！玫瑰为什么是红色的这个问题的答案有点复杂，但我会尽力以简单和礼貌的方式解释。\n玫瑰之所以是红色是因为存在一种叫做花青素的色素。花青素是一种类黄酮类化合物，是许多水果和蔬菜中红色、紫色和蓝色的颜色的来源。\n现在，你可能会想知道为什么玫瑰具体有这种色素。答案在于玫瑰的进化历史。你看，玫瑰已经存在了数百万年，它们的红色很可能在吸引蜜蜂和蝴蝶等传粉者方面发挥了关键作用。这些传粉者被玫瑰鲜艳的颜色所吸引，有助于植物繁殖和传播种子。\n因此，总结一下，玫瑰之所以是红色是因为花青素色素，这是数百万年的进化压力塑造了植物颜色以吸引传粉者。希望这有助于澄清问题。"
```
```python
# 使用流式处理
for chunk in llm.stream("为什么天空是蓝色的？"):
    print(chunk, end=" | ", flush=True)
```
```output
啊 | ， | 一个 | 优秀 | 的 | 问题 | ， | 亲爱 | 的 | 人类 | ！ | * | 调 | 整 | 眼镜 | * | 天空 | 看起来 | 是 | 蓝色 | 是因为 | 一种 | 称为 | 雷利 | 散射 | 的 | 现象 | 。 | 当 | 太阳光 | 进入 | 地球 | 的 | 大气层 | 时 | ， | 它 | 遇到 | 了 | 氮气 | 和 | 氧气 | 等 | 气体 | 的 | 微小 | 分子 | 。 | 这些 | 分子 | 会 | 将 | 光 | 向 | 所有 | 方向 | 散射 | ， | 但 | 它们 | 会 | 比 | 更长 | 的 | （ | 红色 | ） | 波长 | 更 | 多 | 地 | 散射 | 较短 | 的 | （ | 蓝色 | ） | 波长 | 。 | 这 | 就是 | 雷利 | 散射 | 。 | 
| 结果 | ， | 蓝光 | 在 | 大气层 | 中 | 散布 | ， | 赋予 | 天空 | 其 | 特有 | 的 | 蓝色 | 色调 | 。 | 蓝光 | 在 | 日出 | 和 | 日落 | 时 | 更 | 明显 | ， | 这是 | 由于 | 大气层 | 在 | 这些 | 时候 | 散射 | 光线 | 。 || 希望 | 这个 | 解释 | 对 | 你 | 有所 | 帮助 | ， | 亲爱 | 的 | 人类 | ！ | 你 | 还 | 想 | 知道 | 其他 | 什么 | 吗 | ？ | * | 微笑 | * | * |  |
```