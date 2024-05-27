

# Databricks

[Databricks](https://www.databricks.com/) Lakehouse 平台将数据、分析和人工智能统一到一个平台上。

这个示例笔记展示了如何将 Databricks 端点包装成 LangChain 中的 LLMs。

支持两种端点类型：

- 服务端点，推荐用于生产和开发，

- 集群驱动程序代理应用，推荐用于交互式开发。

## 安装

在运行此笔记中的代码之前，需要 `mlflow >= 2.9`。如果尚未安装，请使用以下命令进行安装：

```
pip install mlflow>=2.9
```

此外，我们还需要 `dbutils` 用于此示例。

```
pip install dbutils
```

## 包装服务端点：外部模型

先决条件：将 OpenAI API 密钥注册为一个秘密：

  ```bash
  databricks secrets create-scope <scope>
  databricks secrets put-secret <scope> openai-api-key --string-value $OPENAI_API_KEY
  ```

以下代码使用 OpenAI 的 GPT-4 模型创建一个新的服务端点，用于聊天，并使用端点生成响应。

```python
from langchain_community.chat_models import ChatDatabricks
from langchain_core.messages import HumanMessage
from mlflow.deployments import get_deploy_client
client = get_deploy_client("databricks")
secret = "secrets/<scope>/openai-api-key"  # 将 `<scope>` 替换为您的范围
name = "my-chat"  # 如果 my-chat 已存在，请重新命名
client.create_endpoint(
    name=name,
    config={
        "served_entities": [
            {
                "name": "my-chat",
                "external_model": {
                    "name": "gpt-4",
                    "provider": "openai",
                    "task": "llm/v1/chat",
                    "openai_config": {
                        "openai_api_key": "{{" + secret + "}}",
                    },
                },
            }
        ],
    },
)
chat = ChatDatabricks(
    target_uri="databricks",
    endpoint=name,
    temperature=0.1,
)
chat([HumanMessage(content="hello")])
```

```output
content='Hello! How can I assist you today?'
```

## 包装服务端点：基础模型

以下代码使用 `databricks-bge-large-en` 服务端点（无需创建端点）从输入文本生成嵌入向量。

```python
from langchain_community.embeddings import DatabricksEmbeddings
embeddings = DatabricksEmbeddings(endpoint="databricks-bge-large-en")
embeddings.embed_query("hello")[:3]
```

```output
[0.051055908203125, 0.007221221923828125, 0.003879547119140625]
```

## 包装服务端点：自定义模型

先决条件：

- 已注册并部署了一个 LLM 到 [Databricks 服务端点](https://docs.databricks.com/machine-learning/model-serving/index.html)。

- 您对端点具有 ["Can Query" 权限](https://docs.databricks.com/security/auth-authz/access-control/serving-endpoint-acl.html)。

预期的 MLflow 模型签名为：

  - 输入：`[{"name": "prompt", "type": "string"}, {"name": "stop", "type": "list[string]"}]`

  - 输出：`[{"type": "string"}]`

如果模型签名不兼容或您想插入额外配置，可以相应地设置 `transform_input_fn` 和 `transform_output_fn`。

```python
from langchain_community.llms import Databricks
# 如果在“单用户”或“无隔离共享”模式下运行连接到交互式集群的 Databricks 笔记本，
# 您只需要指定端点名称即可创建一个 `Databricks` 实例，以查询同一工作区中的服务端点。
llm = Databricks(endpoint_name="dolly")
llm("How are you?")
```

```output
'I am happy to hear that you are in good health and as always, you are appreciated.'
```

```python
llm("How are you?", stop=["."])
```

```output
'Good'
```

```python
# 否则，您可以手动指定 Databricks 工作区主机名和个人访问令牌
# 或分别设置 `DATABRICKS_HOST` 和 `DATABRICKS_TOKEN` 环境变量。
# 请参阅 https://docs.databricks.com/dev-tools/auth.html#databricks-personal-access-tokens
# 我们强烈建议不要在笔记本中明确暴露 API 令牌。
# 您可以使用 Databricks 密钥管理器安全地存储您的 API 令牌。
# 请参阅 https://docs.databricks.com/dev-tools/databricks-utils.html#secrets-utility-dbutilssecrets
import os
import dbutils
os.environ["DATABRICKS_TOKEN"] = dbutils.secrets.get("myworkspace", "api_token")
llm = Databricks(host="myworkspace.cloud.databricks.com", endpoint_name="dolly")
llm("How are you?")
```

```output
'I am fine. Thank you!'
```

```python
# 如果服务端点接受额外参数，如 `temperature`，
# 您可以在 `model_kwargs` 中设置它们。
llm = Databricks(endpoint_name="dolly", model_kwargs={"temperature": 0.1})
llm("How are you?")
```

```output
'I am fine.'
```

```python
# 如果服务端点期望不同的输入模式并且不返回 JSON 字符串，
# 或者您想在其上应用提示模板，则可以使用 `transform_input_fn` 和 `transform_output_fn`。
def transform_input(**request):
    full_prompt = f"""{request["prompt"]}
    Be Concise.
    """
    request["prompt"] = full_prompt
    return request
llm = Databricks(endpoint_name="dolly", transform_input_fn=transform_input)
llm("How are you?")
```

## 包装集群驱动代理应用程序

先决条件：

* 在“单用户”或“无隔离共享”模式下加载在 Databricks 交互式集群上的 LLM。

* 在驱动节点上运行一个本地 HTTP 服务器，使用 HTTP POST 在 `"/"` 上提供模型，输入/输出为 JSON。

* 使用端口号在 `[3000, 8000]` 之间，并侦听驱动器 IP 地址或简单地使用 `0.0.0.0` 而不是仅限本地主机。

* 您具有对集群的“Can Attach To”权限。

预期的服务器模式（使用 JSON 模式）为：

* 输入：

  ```json
  {"type": "object",
   "properties": {
      "prompt": {"type": "string"},
       "stop": {"type": "array", "items": {"type": "string"}}},
    "required": ["prompt"]}
  ```

* 输出：`{"type": "string"}`

如果服务器模式不兼容或您想要插入额外配置，可以相应地使用 `transform_input_fn` 和 `transform_output_fn`。

以下是运行驱动代理应用程序以提供 LLM 的最小示例：

```python
from flask import Flask, request, jsonify
import torch
from transformers import pipeline, AutoTokenizer, StoppingCriteria
model = "databricks/dolly-v2-3b"
tokenizer = AutoTokenizer.from_pretrained(model, padding_side="left")
dolly = pipeline(model=model, tokenizer=tokenizer, trust_remote_code=True, device_map="auto")
device = dolly.device
class CheckStop(StoppingCriteria):
    def __init__(self, stop=None):
        super().__init__()
        self.stop = stop or []
        self.matched = ""
        self.stop_ids = [tokenizer.encode(s, return_tensors='pt').to(device) for s in self.stop]
    def __call__(self, input_ids: torch.LongTensor, scores: torch.FloatTensor, **kwargs):
        for i, s in enumerate(self.stop_ids):
            if torch.all((s == input_ids[0][-s.shape[1]:])).item():
                self.matched = self.stop[i]
                return True
        return False
def llm(prompt, stop=None, **kwargs):
  check_stop = CheckStop(stop)
  result = dolly(prompt, stopping_criteria=[check_stop], **kwargs)
  return result[0]["generated_text"].rstrip(check_stop.matched)
app = Flask("dolly")
@app.route('/', methods=['POST'])
def serve_llm():
  resp = llm(**request.json)
  return jsonify(resp)
app.run(host="0.0.0.0", port="7777")
```

一旦服务器运行，您可以创建一个 `Databricks` 实例来将其包装为 LLM。

```python
# 如果在运行应用程序的集群上附加了与运行应用程序相同的集群的 Databricks 笔记本，
# 您只需要指定驱动器端口来创建一个 `Databricks` 实例。
llm = Databricks(cluster_driver_port="7777")
llm("你好吗？")
```

```output
'你好，谢谢你的关心。听到你过得很好真是太棒了。'
```

```python
# 否则，您可以手动指定要使用的集群 ID，
# 以及 Databricks 工作区主机名和个人访问令牌。
llm = Databricks(cluster_id="0000-000000-xxxxxxxx", cluster_driver_port="7777")
llm("你好吗？")
```

```output
'我很好。你呢？'
```

```python
# 如果应用程序接受额外参数，比如 `temperature`，
# 您可以在 `model_kwargs` 中设置它们。
llm = Databricks(cluster_driver_port="7777", model_kwargs={"temperature": 0.1})
llm("你好吗？")
```

```output
'我很好。很高兴见到你。'
```

```python
# 如果应用程序期望不同的输入模式并且不返回 JSON 字符串，
# 或者您想在其上应用提示模板，则使用 `transform_input_fn` 和 `transform_output_fn`。
def transform_input(**request):
    full_prompt = f"""{request["prompt"]}
    言简意赅。
    """
    request["prompt"] = full_prompt
    return request
def transform_output(response):
    return response.upper()
llm = Databricks(
    cluster_driver_port="7777",
    transform_input_fn=transform_input,
    transform_output_fn=transform_output,
)
llm("你好吗？")
```

```output
'我很好，谢谢你。'
```

