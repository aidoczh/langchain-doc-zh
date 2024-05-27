# 阿列夫·阿尔法

[光辉系列](https://docs.aleph-alpha.com/docs/introduction/luminous/)是一系列大型语言模型。

这个示例演示了如何使用 LangChain 与阿列夫·阿尔法模型进行交互

```python
# 安装包
%pip install --upgrade --quiet  aleph-alpha-client
```

```python
# 创建一个新的令牌：https://docs.aleph-alpha.com/docs/account/#create-a-new-token
from getpass import getpass
ALEPH_ALPHA_API_KEY = getpass()
```

```output
········
```

```python
from langchain_community.llms import AlephAlpha
from langchain_core.prompts import PromptTemplate
```

```python
template = """Q: {question}
A:"""
prompt = PromptTemplate.from_template(template)
```

```python
llm = AlephAlpha(
    model="luminous-extended",
    maximum_tokens=20,
    stop_sequences=["Q:"],
    aleph_alpha_api_key=ALEPH_ALPHA_API_KEY,
)
```

```python
llm_chain = prompt | llm
```

```python
question = "What is AI?"
llm_chain.invoke({"question": question})
```

```output
'人工智能是机器模拟人类智能过程。\n\n'
```