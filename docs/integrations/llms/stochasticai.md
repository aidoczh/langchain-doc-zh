# StochasticAI

>[Stochastic Acceleration Platform](https://docs.stochastic.ai/docs/introduction/) 旨在简化深度学习模型的生命周期。从上传和版本控制模型，到训练、压缩和加速，再到投入生产。

本示例介绍如何使用 LangChain 与 `StochasticAI` 模型进行交互。

您需要在[此处](https://app.stochastic.ai/workspace/profile/settings?tab=profile)获取 API_KEY 和 API_URL。

```python
from getpass import getpass
STOCHASTICAI_API_KEY = getpass()
```

```output
 ········
```

```python
import os
os.environ["STOCHASTICAI_API_KEY"] = STOCHASTICAI_API_KEY
```

```python
YOUR_API_URL = getpass()
```

```output
 ········
```

```python
from langchain.chains import LLMChain
from langchain_community.llms import StochasticAI
from langchain_core.prompts import PromptTemplate
```

```python
template = """问题: {question}
回答: 让我们逐步思考。"""
prompt = PromptTemplate.from_template(template)
```

```python
llm = StochasticAI(api_url=YOUR_API_URL)
```

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

```python
question = "贾斯汀·比伯出生的那一年，哪支美式足球队赢得了超级碗冠军？"
llm_chain.run(question)
```

```output
"\n\n第一步: 1999年，圣路易斯公羊队赢得了超级碗冠军。\n\n第二步: 1999年，比伯出生。\n\n第三步: 当时公羊队在洛杉矶。\n\n第四步: 所以他们那年没有参加超级碗。\n"
```