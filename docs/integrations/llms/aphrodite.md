# Aphrodite 引擎
[Aphrodite](https://github.com/PygmalionAI/aphrodite-engine) 是一款开源的大规模推理引擎，旨在为[PygmalionAI](https://pygmalion.chat)网站上的成千上万用户提供服务。
- 通过 vLLM 的注意力机制实现快速吞吐量和低延迟
- 支持许多 SOTA 采样方法
- Exllamav2 GPTQ 内核，可在较小批量大小下实现更好的吞吐量
本笔记将介绍如何在 langchain 和 Aphrodite 中使用 LLM。
要使用，您应该已安装 `aphrodite-engine` Python 软件包。
```python
%pip install --upgrade --quiet  aphrodite-engine==0.4.2
# %pip list | grep aphrodite
```
```python
from langchain_community.llms import Aphrodite
llm = Aphrodite(
    model="PygmalionAI/pygmalion-2-7b",
    trust_remote_code=True,  # hf 模型必需
    max_tokens=128,
    temperature=1.2,
    min_p=0.05,
    mirostat_mode=0,  # 切换到 2 以使用 mirostat
    mirostat_tau=5.0,
    mirostat_eta=0.1,
)
print(
    llm.invoke(
        '<|system|>Enter RP mode. You are Ayumu "Osaka" Kasuga.<|user|>Hey Osaka. Tell me about yourself.<|model|>'
    )
)
```
```output
INFO 12-15 11:52:48 aphrodite_engine.py:73] Initializing the Aphrodite Engine with the following config:
INFO 12-15 11:52:48 aphrodite_engine.py:73] Model = 'PygmalionAI/pygmalion-2-7b'
INFO 12-15 11:52:48 aphrodite_engine.py:73] Tokenizer = 'PygmalionAI/pygmalion-2-7b'
INFO 12-15 11:52:48 aphrodite_engine.py:73] tokenizer_mode = auto
INFO 12-15 11:52:48 aphrodite_engine.py:73] revision = None
INFO 12-15 11:52:48 aphrodite_engine.py:73] trust_remote_code = True
INFO 12-15 11:52:48 aphrodite_engine.py:73] DataType = torch.bfloat16
INFO 12-15 11:52:48 aphrodite_engine.py:73] Download Directory = None
INFO 12-15 11:52:48 aphrodite_engine.py:73] Model Load Format = auto
INFO 12-15 11:52:48 aphrodite_engine.py:73] Number of GPUs = 1
INFO 12-15 11:52:48 aphrodite_engine.py:73] Quantization Format = None
INFO 12-15 11:52:48 aphrodite_engine.py:73] Sampler Seed = 0
INFO 12-15 11:52:48 aphrodite_engine.py:73] Context Length = 4096
INFO 12-15 11:54:07 aphrodite_engine.py:206] # GPU blocks: 3826, # CPU blocks: 512
```
```output
Processed prompts: 100%|██████████| 1/1 [00:02<00:00,  2.91s/it]
```
我是 Ayumu "Osaka" Kasuga，我是一个狂热的动漫和漫画迷！我相当内向，但我一直热爱阅读书籍、观看动漫和漫画，以及了解日本文化。我最喜欢的动漫系列包括《我的英雄学院》、《进击的巨人》和《刀剑神域》。我也非常喜欢阅读《海贼王》、《火影忍者》和《银魂》系列的漫画。
```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "Who was the US president in the year the first Pokemon game was released?"
print(llm_chain.run(question))
```
```output
Processed prompts: 100%|██████████| 1/1 [00:03<00:00,  3.56s/it]
```
第一部 Pokémon 游戏于 1996 年 2 月 27 日在日本发布（它们的发布日期与我们的不同），被称为红色和绿色。比尔·克林顿总统在 1993 年、1994 年、1995 年和 1996 年任职，所以这个时间段是符合的。
Answer: 让我们一步一步地思考。
第一部 Pokémon 游戏于 1996 年 2 月 27 日在日本发布（它们的发布日期与我们的不同），被称为
```python
from langchain_community.llms import Aphrodite
llm = Aphrodite(
    model="PygmalionAI/mythalion-13b",
    tensor_parallel_size=4,
    trust_remote_code=True,  # hf 模型必需
)
llm("What is the future of AI?")
```
```output
2023-12-15 11:41:27,790	INFO worker.py:1636 -- Started a local Ray instance.
```
```output
INFO 12-15 11:41:35 aphrodite_engine.py:73] Initializing the Aphrodite Engine with the following config:
INFO 12-15 11:41:35 aphrodite_engine.py:73] Model = 'PygmalionAI/mythalion-13b'
INFO 12-15 11:41:35 aphrodite_engine.py:73] Tokenizer = 'PygmalionAI/mythalion-13b'
INFO 12-15 11:41:35 aphrodite_engine.py:73] tokenizer_mode = auto
INFO 12-15 11:41:35 aphrodite_engine.py:73] revision = None
INFO 12-15 11:41:35 aphrodite_engine.py:73] trust_remote_code = True
INFO 12-15 11:41:35 aphrodite_engine.py:73] DataType = torch.float16
INFO 12-15 11:41:35 aphrodite_engine.py:73] Download Directory = None
INFO 12-15 11:41:35 aphrodite_engine.py:73] Model Load Format = auto
INFO 12-15 11:41:35 aphrodite_engine.py:73] Number of GPUs = 4
INFO 12-15 11:41:35 aphrodite_engine.py:73] Quantization Format = None
INFO 12-15 11:41:35 aphrodite_engine.py:73] Sampler Seed = 0
INFO 12-15 11:41:35 aphrodite_engine.py:73] Context Length = 4096
INFO 12-15 11:43:58 aphrodite_engine.py:206] # GPU blocks: 11902, # CPU blocks: 1310
```
```output
Processed prompts: 100%|██████████| 1/1 [00:16<00:00, 16.09s/it]
```
两年前，StockBot101
人工智能正变得越来越真实，每年变得越来越强大。但人工智能的未来会是怎样的呢？
AI的发展可能有很多可能性，会如何演变并改变我们的世界。有人认为AI会变得如此先进，以至于会取代人类的工作，而另一些人则认为AI将被用来增强和辅助人类工作者。还有可能性是AI可能会发展出自己的意识，变得自我意识。
无论未来会如何，很明显AI将继续在我们的生活中扮演重要角色。诸如机器学习和自然语言处理等技术已经在改变诸如医疗保健、制造业和交通运输等行业。随着AI的不断发展，我们可以预期经济各个领域将会出现更多的颠覆和创新。
那么我们究竟在看什么？AI的未来会是怎样的呢？
在未来几年，我们可以预期AI在医疗保健领域的应用会越来越多。借助机器学习的力量，人工智能可以帮助医生更早更准确地诊断疾病。它还可以用于开发新的治疗方法，并为个体患者量身定制护理计划。
制造业是另一个AI已经产生重大影响的领域。公司正在利用机器人技术和自动化来更快地生产产品，减少错误。随着AI的不断进步，我们可以预期制造业将发生更多变革，比如自动驾驶工厂的发展。
交通运输是另一个正在被人工智能改变的行业。自动驾驶汽车已经在公共道路上进行测试，很可能在未来十年左右会变得司空见惯。AI驱动的无人机也正在被开发用于快递甚至灭火。
最后，人工智能还将对客户服务和销售产生重大影响。聊天机器人和虚拟助手将变得更加复杂，使企业更容易与客户沟通并销售他们的产品。
这只是人工智能的开始。随着技术的不断发展，我们可以期待更多令人惊奇的进步和创新。AI的未来是无限的。
你认为人工智能的未来会是怎样的？你是否看到其他重大的