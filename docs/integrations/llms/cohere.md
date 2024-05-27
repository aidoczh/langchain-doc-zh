# Cohere

[Cohere](https://cohere.ai/about) 是一家加拿大初创公司，提供自然语言处理模型，帮助公司改进人机交互。

查看 [API 参考文档](https://api.python.langchain.com/en/latest/llms/langchain_community.llms.cohere.Cohere.html) 以获取所有属性和方法的详细文档。

## 设置

集成位于 `langchain-community` 包中。我们还需要安装 `cohere` 包本身。可以使用以下命令安装：

```bash
pip install -U langchain-community langchain-cohere
```

我们还需要获取 [Cohere API 密钥](https://cohere.com/) 并设置 `COHERE_API_KEY` 环境变量：

```python
import getpass
import os
os.environ["COHERE_API_KEY"] = getpass.getpass()
```

```output
 ········
```

设置 [LangSmith](https://smith.langchain.com/) 以获得最佳的可观察性（可选）

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 用法

Cohere 支持所有 [LLM](/docs/how_to#llms) 功能：

```python
from langchain_cohere import Cohere
from langchain_core.messages import HumanMessage
```

```python
model = Cohere(model="command", max_tokens=256, temperature=0.75)
```

```python
message = "敲敲"
model.invoke(message)
```

```output
" 谁在敲门？"
```

```python
await model.ainvoke(message)
```

```output
" 谁在敲门？"
```

```python
for chunk in model.stream(message):
    print(chunk, end="", flush=True)
```

```output
 谁在敲门？
```

```python
model.batch([message])
```

```output
[" 谁在敲门？"]
```

您还可以轻松地与提示模板结合，以便轻松构建用户输入的结构。我们可以使用 [LCEL](/docs/concepts#langchain-expression-language) 来实现这一点：

```python
from langchain_core.prompts import PromptTemplate
prompt = PromptTemplate.from_template("告诉我一个关于 {topic} 的笑话")
chain = prompt | model
```

```python
chain.invoke({"topic": "熊"})
```

```output
' 为什么泰迪熊要过马路？\n因为他有熊过街。\n\n您想听另一个笑话吗？ '
```