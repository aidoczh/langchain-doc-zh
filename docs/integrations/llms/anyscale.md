# Anyscale

[Anyscale](https://www.anyscale.com/) 是一个完全托管的 [Ray](https://www.ray.io/) 平台，您可以在其上构建、部署和管理可扩展的人工智能和 Python 应用程序。

这个示例演示了如何使用 LangChain 与 [Anyscale Endpoint](https://app.endpoints.anyscale.com/) 进行交互。

```python
ANYSCALE_API_BASE = "..."
ANYSCALE_API_KEY = "..."
ANYSCALE_MODEL_NAME = "..."
```

```python
import os
os.environ["ANYSCALE_API_BASE"] = ANYSCALE_API_BASE
os.environ["ANYSCALE_API_KEY"] = ANYSCALE_API_KEY
```

```python
from langchain.chains import LLMChain
from langchain_community.llms import Anyscale
from langchain_core.prompts import PromptTemplate
```

```python
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
```

```python
llm = Anyscale(model_name=ANYSCALE_MODEL_NAME)
```

```python
llm_chain = prompt | llm
```

```python
question = "When was George Washington president?"
llm_chain.invoke({"question": question})
```

使用 Ray，我们可以在没有异步实现的情况下分发查询。这不仅适用于 Anyscale LLM 模型，还适用于任何其他 Langchain LLM 模型，这些模型没有实现 `_acall` 或 `_agenerate`。

```python
prompt_list = [
    "When was George Washington president?",
    "Explain to me the difference between nuclear fission and fusion.",
    "Give me a list of 5 science fiction books I should read next.",
    "Explain the difference between Spark and Ray.",
    "Suggest some fun holiday ideas.",
    "Tell a joke.",
    "What is 2+2?",
    "Explain what is machine learning like I am five years old.",
    "Explain what is artifical intelligence.",
]
```

```python
import ray
@ray.remote(num_cpus=0.1)
def send_query(llm, prompt):
    resp = llm.invoke(prompt)
    return resp
futures = [send_query.remote(llm, prompt) for prompt in prompt_list]
results = ray.get(futures)
```