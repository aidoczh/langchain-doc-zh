# Writer

[Writer](https://writer.com/) 是一个生成不同语言内容的平台。

以下示例介绍如何使用 LangChain 与 `Writer` [模型](https://dev.writer.com/docs/models) 进行交互。

你需要在[这里](https://dev.writer.com/docs)获取 WRITER_API_KEY。

```python
from getpass import getpass
WRITER_API_KEY = getpass()
```

```output
 ········
```

```python
import os
os.environ["WRITER_API_KEY"] = WRITER_API_KEY
```

```python
from langchain.chains import LLMChain
from langchain_community.llms import Writer
from langchain_core.prompts import PromptTemplate
```

```python
template = """问题: {question}
答案: 让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
```

```python
# 如果出现错误，可能需要设置 "base_url" 参数，该参数可以从错误日志中获取。
llm = Writer()
```

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

```python
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.run(question)
```