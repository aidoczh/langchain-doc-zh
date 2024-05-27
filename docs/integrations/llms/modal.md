# 模态

[Modal 云平台](https://modal.com/docs/guide) 提供了方便的按需访问，可以从您的本地计算机上的 Python 脚本运行无服务器云计算。使用 `modal` 运行您自己的自定义 LLM 模型，而不是依赖于 LLM API。

这个示例介绍了如何使用 LangChain 与 `modal` 的 HTTPS [web 端点](https://modal.com/docs/guide/webhooks) 进行交互。

[_使用 LangChain 进行问答_](https://modal.com/docs/guide/ex/potus_speech_qanda) 是另一个使用 LangChain 与 `Modal` 结合的示例。在该示例中，Modal 完全运行 LangChain 应用程序，并使用 OpenAI 作为其 LLM API。

```python
%pip install --upgrade --quiet  modal
```

```python
# 使用 Modal 注册账户并获取新的令牌。
!modal token new
```

```output
在您的浏览器窗口中启动登录页面...
如果没有显示，请手动将此网址复制到您的浏览器中：
https://modal.com/token-flow/tf-Dzm3Y01234mqmm1234Vcu3
```

[`langchain.llms.modal.Modal`](https://github.com/langchain-ai/langchain/blame/master/langchain/llms/modal.py) 集成类要求您部署一个符合以下 JSON 接口的 Modal 应用程序的 web 端点：

1. LLM 提示作为 `"prompt"` 键下的 `str` 值被接受

2. LLM 响应作为 `"prompt"` 键下的 `str` 值返回

**示例请求 JSON:**

```json
{
    "prompt": "Identify yourself, bot!",
    "extra": "args are allowed",
}
```

**示例响应 JSON:**

```json
{
    "prompt": "This is the LLM speaking",
}
```

一个满足这一接口的示例 '虚拟' Modal web 端点函数可能是

```python
...
...
class Request(BaseModel):
    prompt: str
@stub.function()
@modal.web_endpoint(method="POST")
def web(request: Request):
    _ = request  # 忽略输入
    return {"prompt": "hello world"}
```

* 请参阅 Modal 的 [web 端点](https://modal.com/docs/guide/webhooks#passing-arguments-to-web-endpoints) 指南，了解设置符合此接口的端点的基础知识。

* 请参阅 Modal 的 ['使用 AutoGPTQ 运行 Falcon-40B'](https://modal.com/docs/guide/ex/falcon_gptq) 开源 LLM 示例，作为自定义 LLM 的起点！

一旦您部署了 Modal web 端点，您就可以将其 URL 传递给 `langchain.llms.modal.Modal` LLM 类。然后，该类可以作为您链中的一个构建模块。

```python
from langchain.chains import LLMChain
from langchain_community.llms import Modal
from langchain_core.prompts import PromptTemplate
```

```python
template = """问题: {question}
答案: 让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
```

```python
endpoint_url = "https://ecorp--custom-llm-endpoint.modal.run"  # 用您部署的 Modal web 端点的 URL 替换此处
llm = Modal(endpoint_url=endpoint_url)
```

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

```python
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.run(question)
```