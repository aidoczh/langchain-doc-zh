# C 变压器

[C 变压器](https://github.com/marella/ctransformers) 库为 GGML 模型提供了 Python 绑定。

这个示例演示了如何使用 LangChain 与 `C 变压器` [模型](https://github.com/marella/ctransformers#supported-models) 进行交互。

**安装**

```python
%pip install --upgrade --quiet  ctransformers
```

**加载模型**

```python
from langchain_community.llms import CTransformers
llm = CTransformers(model="marella/gpt-2-ggml")
```

**生成文本**

```python
print(llm.invoke("AI is going to"))
```

**流式处理**

```python
from langchain_core.callbacks import StreamingStdOutCallbackHandler
llm = CTransformers(
    model="marella/gpt-2-ggml", callbacks=[StreamingStdOutCallbackHandler()]
)
response = llm.invoke("AI is going to")
```

**LLMChain**

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
template = """Question: {question}
Answer:"""
prompt = PromptTemplate.from_template(template)
llm_chain = LLMChain(prompt=prompt, llm=llm)
response = llm_chain.run("What is AI?")
```