# TextGen

[GitHub:oobabooga/text-generation-webui](https://github.com/oobabooga/text-generation-webui) 是一个 Gradio 网页用户界面，用于运行大型语言模型，如 LLaMA、llama.cpp、GPT-J、Pythia、OPT 和 GALACTICA。

这个示例介绍了如何使用 LangChain 与 LLM 模型进行交互，通过 `text-generation-webui` 的 API 集成。

请确保已经配置好 `text-generation-webui` 并安装了 LLM。建议通过适用于您操作系统的 [一键安装程序](https://github.com/oobabooga/text-generation-webui#one-click-installers) 进行安装。

一旦安装并通过网页界面确认 `text-generation-webui` 正常工作，请通过网页模型配置选项启用 `api` 选项，或者在启动命令中添加运行时参数 `--api`。

## 设置 model_url 并运行示例

```python
model_url = "http://localhost:5000"
```

```python
from langchain.chains import LLMChain
from langchain.globals import set_debug
from langchain_community.llms import TextGen
from langchain_core.prompts import PromptTemplate
set_debug(True)
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
llm = TextGen(model_url=model_url)
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "What NFL team won the Super Bowl in the year Justin Bieber was born?"
llm_chain.run(question)
```

### 流式版本

您应该安装 websocket-client 以使用此功能。

`pip install websocket-client`

```python
model_url = "ws://localhost:5005"
```

```python
from langchain.chains import LLMChain
from langchain.globals import set_debug
from langchain_community.llms import TextGen
from langchain_core.callbacks import StreamingStdOutCallbackHandler
from langchain_core.prompts import PromptTemplate
set_debug(True)
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
llm = TextGen(
    model_url=model_url, streaming=True, callbacks=[StreamingStdOutCallbackHandler()]
)
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "What NFL team won the Super Bowl in the year Justin Bieber was born?"
llm_chain.run(question)
```

```python
llm = TextGen(model_url=model_url, streaming=True)
for chunk in llm.stream("Ask 'Hi, how are you?' like a pirate:'", stop=["'", "\n"]):
    print(chunk, end="", flush=True)
```