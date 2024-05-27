# Ray Serve

[Ray Serve](https://docs.ray.io/en/latest/serve/index.html) 是一个可扩展的模型服务库，用于构建在线推理 API。Serve 特别适用于系统组合，使您能够在 Python 代码中构建由多个链和业务逻辑组成的复杂推理服务。

## 本笔记本的目标

本笔记本展示了如何将 OpenAI 链部署到生产环境的简单示例。您可以扩展它，以部署您自己的自托管模型，在其中可以轻松定义在生产环境中运行模型所需的硬件资源（GPU 和 CPU）的数量。更多可用选项，包括自动缩放，请阅读 Ray Serve 的[文档](https://docs.ray.io/en/latest/serve/getting_started.html)。

## 设置 Ray Serve

使用 `pip install ray[serve]` 安装 ray。

## 通用框架

部署服务的通用框架如下：

```python
# 0: 导入 ray serve 和 starlette 的 request
from ray import serve
from starlette.requests import Request
# 1: 定义一个 Ray Serve 部署
@serve.deployment
class LLMServe:
    def __init__(self) -> None:
        # 所有的初始化代码都在这里
        pass
    async def __call__(self, request: Request) -> str:
        # 您可以在这里解析请求
        # 并返回一个响应
        return "Hello World"
# 2: 将模型绑定到部署
deployment = LLMServe.bind()
# 3: 运行部署
serve.api.run(deployment)
```

```python
# 关闭部署
serve.api.shutdown()
```

## 部署和 OpenAI 链的示例

从[这里](https://platform.openai.com/account/api-keys)获取 OpenAI API 密钥。通过运行以下代码，您将被要求提供 API 密钥。

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
```

```python
from getpass import getpass
OPENAI_API_KEY = getpass()
```

```python
@serve.deployment
class DeployLLM:
    def __init__(self):
        # 我们在这里初始化 LLM、模板和链
        llm = OpenAI(openai_api_key=OPENAI_API_KEY)
        template = "Question: {question}\n\nAnswer: Let's think step by step."
        prompt = PromptTemplate.from_template(template)
        self.chain = LLMChain(llm=llm, prompt=prompt)
    def _run_chain(self, text: str):
        return self.chain(text)
    async def __call__(self, request: Request):
        # 1. 解析请求
        text = request.query_params["text"]
        # 2. 运行链
        resp = self._run_chain(text)
        # 3. 返回响应
        return resp["text"]
```

现在我们可以将部署绑定起来。

```python
# 将模型绑定到部署
deployment = DeployLLM.bind()
```

当我们想要运行部署时，我们可以指定端口号和主机。

```python
# 示例端口号
PORT_NUMBER = 8282
# 运行部署
serve.api.run(deployment, port=PORT_NUMBER)
```

现在服务部署在 `localhost:8282` 端口上，我们可以发送一个 POST 请求来获取结果。

```python
import requests
text = "What NFL team won the Super Bowl in the year Justin Beiber was born?"
response = requests.post(f"http://localhost:{PORT_NUMBER}/?text={text}")
print(response.content.decode())
```