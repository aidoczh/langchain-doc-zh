# OpenLLM

[🦾 OpenLLM](https://github.com/bentoml/OpenLLM) 是一个用于在生产环境中操作大型语言模型（LLMs）的开放平台。它使开发人员能够轻松地运行任何开源LLMs的推理，部署到云端或本地，并构建强大的人工智能应用程序。

## 安装

通过 [PyPI](https://pypi.org/project/openllm/) 安装 `openllm`

```python
%pip install --upgrade --quiet  openllm
```

## 在本地启动 OpenLLM 服务器

要启动一个LLM服务器，请使用 `openllm start` 命令。例如，要启动一个dolly-v2服务器，请从终端运行以下命令：

```bash
openllm start dolly-v2
```

## 包装器

```python
from langchain_community.llms import OpenLLM
server_url = "http://localhost:3000"  # 如果在远程服务器上运行，请替换为远程主机
llm = OpenLLM(server_url=server_url)
```

### 可选：本地LLM推理

您也可以选择从当前进程中本地初始化由OpenLLM管理的LLM。这对于开发目的很有用，并允许开发人员快速尝试不同类型的LLMs。

当将LLM应用程序移至生产环境时，我们建议单独部署OpenLLM服务器，并通过上面演示的 `server_url` 选项进行访问。

要通过LangChain包装器本地加载LLM：

```python
from langchain_community.llms import OpenLLM
llm = OpenLLM(
    model_name="dolly-v2",
    model_id="databricks/dolly-v2-3b",
    temperature=0.94,
    repetition_penalty=1.2,
)
```

### 与LLMChain集成

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
template = "一个制造{product}的公司应该取什么好名字？"
prompt = PromptTemplate.from_template(template)
llm_chain = LLMChain(prompt=prompt, llm=llm)
generated = llm_chain.run(product="机械键盘")
print(generated)
```

```output
iLkb
```