# 极小化

[Minimax](https://api.minimax.chat) 是一家中国初创公司，为企业和个人提供自然语言处理模型。

这个示例演示了如何使用 Langchain 与 Minimax 进行交互。

# 设置

要运行这个笔记本，您需要一个 [Minimax 账户](https://api.minimax.chat)，一个 [API 密钥](https://api.minimax.chat/user-center/basic-information/interface-key)，以及一个 [群组 ID](https://api.minimax.chat/user-center/basic-information)。

# 单模型调用

```python
from langchain_community.llms import Minimax
```

```python
# 加载模型
minimax = Minimax(minimax_api_key="YOUR_API_KEY", minimax_group_id="YOUR_GROUP_ID")
```

```python
# 提示模型
minimax("熊猫和熊之间有什么区别？")
```

# 链式模型调用

```python
# 获取 api_key 和 group_id: https://api.minimax.chat/user-center/basic-information
# 我们需要 `MINIMAX_API_KEY` 和 `MINIMAX_GROUP_ID`
import os
os.environ["MINIMAX_API_KEY"] = "YOUR_API_KEY"
os.environ["MINIMAX_GROUP_ID"] = "YOUR_GROUP_ID"
```

```python
from langchain.chains import LLMChain
from langchain_community.llms import Minimax
from langchain_core.prompts import PromptTemplate
```

```python
template = """问题: {question}
答案: 让我们逐步思考。"""
prompt = PromptTemplate.from_template(template)
```

```python
llm = Minimax()
```

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

```python
question = "周杰伦出生年份哪一年，NBA 哪支球队赢得了总冠军？"
llm_chain.run(question)
```