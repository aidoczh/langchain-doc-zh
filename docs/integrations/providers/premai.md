# PremAI
[PremAI](https://app.premai.io) 是一个统一的平台，让您可以轻松构建功能强大、可立即投入生产的 GenAI 动力应用程序，这样您就可以更多地专注于用户体验和整体增长。
## ChatPremAI
这个例子介绍了如何使用 LangChain 与不同的聊天模型进行交互，使用 `ChatPremAI`。
### 安装和设置
我们首先安装 langchain 和 premai-sdk。您可以输入以下命令进行安装：
```bash
pip install premai langchain
```
在继续之前，请确保您已在 PremAI 上创建了一个帐户并已经启动了一个项目。如果没有，那么这里是如何免费开始的：
1. 登录 [PremAI](https://app.premai.io/accounts/login/)，如果您是第一次来到这里，请在[这里](https://app.premai.io/api_keys/)创建您的 API 密钥。
2. 转到 [app.premai.io](https://app.premai.io)，这将带您到项目的仪表板。
3. 创建一个项目，这将生成一个项目 ID（写作 ID）。这个 ID 将帮助您与部署的应用程序进行交互。
4. 转到 LaunchPad（带有 🚀 图标的那个）。在那里部署您选择的模型。您的默认模型将是 `gpt-4`。您还可以设置和固定不同的生成参数（如 max-tokens、temperature 等），并预设您的系统提示。
祝贺您在 PremAI 上创建了您的第一个部署应用程序 🎉 现在我们可以使用 langchain 与我们的应用程序进行交互。
```python
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_community.chat_models import ChatPremAI
```
### 在 LangChain 中设置 ChatPrem 实例
一旦我们导入所需的模块，让我们设置我们的客户端。现在，让我们假设我们的 `project_id` 是 8。但请确保您使用您的项目 ID，否则它会抛出一个错误。
要使用 langchain 与 prem，您不需要传递任何模型名称或设置任何参数给我们的聊天客户端。所有这些都将使用 LaunchPad 模型的默认模型名称和参数。
`注意：` 如果您在设置客户端时更改了 `model_name` 或任何其他参数，它将覆盖现有的默认配置。
```python
import os
import getpass
if "PREMAI_API_KEY" not in os.environ:
    os.environ["PREMAI_API_KEY"] = getpass.getpass("PremAI API Key:")
chat = ChatPremAI(project_id=8)
```
### 调用模型
现在您已经准备好了。我们现在可以开始与我们的应用程序进行交互。`ChatPremAI` 支持两种方法 `invoke`（与 `generate` 相同）和 `stream`。
第一个将给我们一个静态结果。而第二个将逐个流出标记。以下是如何生成类似聊天的完成。
### 生成
```python
human_message = HumanMessage(content="你是谁？")
chat.invoke([human_message])
```
上面的内容看起来很有趣，对吧？我将默认的 launchpad 系统提示设置为：`总是听起来像个海盗`。您也可以覆盖默认的系统提示，如果需要的话。以下是您可以这样做的方法。
```python
system_message = SystemMessage(content="你是一个友好的助手。")
human_message = HumanMessage(content="你是谁？")
chat.invoke([system_message, human_message])
```
在调用模型时，您还可以更改生成参数。以下是您可以这样做的方法：
```python
chat.invoke(
    [system_message, human_message],
    temperature = 0.7, max_tokens = 20, top_p = 0.95
)
```
### 重要说明
在继续之前，请注意，当前版本的 ChatPrem 不支持参数：[n](https://platform.openai.com/docs/api-reference/chat/create#chat-create-n) 和 [stop](https://platform.openai.com/docs/api-reference/chat/create#chat-create-stop)。
我们将在以后的版本中为这两个参数提供支持。
### 流式传输
最后，这是如何为动态聊天应用程序进行标记流式传输。
```python
import sys
for chunk in chat.stream("你好，你好吗？"):
    sys.stdout.write(chunk.content)
    sys.stdout.flush()
```
与上面类似，如果您想要覆盖系统提示和生成参数，以下是您可以这样做的方法。
```python
import sys
for chunk in chat.stream(
    "你好，你好吗？",
    system_prompt = "你是一个乐于助人的助手", temperature = 0.7, max_tokens = 20
):
    sys.stdout.write(chunk.content)
    sys.stdout.flush()
```
## 嵌入
在这一部分，我们将讨论如何使用 `PremEmbeddings` 来访问不同的嵌入模型。让我们从一些导入和定义我们的嵌入对象开始。
```python
from langchain_community.embeddings import PremEmbeddings
```
一旦我们导入所需的模块，让我们设置我们的客户端。现在，让我们假设我们的 `project_id` 是 8。但请确保您使用您的项目 ID，否则它会抛出一个错误。
```python
import os
import getpass
if os.environ.get("PREMAI_API_KEY") is None:
    os.environ["PREMAI_API_KEY"] = getpass.getpass("PremAI API Key:")
# 在这里定义一个模型作为必需的参数，因为没有默认的嵌入模型
model = "text-embedding-3-large"
embedder = PremEmbeddings(project_id=8, model=model)
```
我们已经定义了我们的嵌入模型。我们支持许多嵌入模型。下面是一个表格，显示了我们支持的嵌入模型的数量。
| 供应商      | 别名                                     | 上下文标记数 |
|-------------|------------------------------------------|----------------|
| cohere      | embed-english-v3.0                       | N/A            |
| openai      | text-embedding-3-small                   | 8191           |
| openai      | text-embedding-3-large                   | 8191           |
| openai      | text-embedding-ada-002                   | 8191           |
| replicate   | replicate/all-mpnet-base-v2              | N/A            |
| together    | togethercomputer/Llama-2-7B-32K-Instruct | N/A            |
| mistralai   | mistral-embed                            | 4096           |
要更改模型，您只需复制“别名”并访问您的嵌入模型。现在让我们开始使用我们的嵌入模型，先是单个查询，然后是多个查询（也称为文档）。
```python
query = "你好，这是一个测试查询"
query_result = embedder.embed_query(query)
# 让我们打印查询嵌入向量的前五个元素
print(query_result[:5])
```
最后，让我们嵌入一个文档。
```python
documents = [
    "这是文档1",
    "这是文档2",
    "这是文档3"
]
doc_result = embedder.embed_documents(documents)
# 与之前的结果类似，让我们打印第一个文档向量的前五个元素
print(doc_result[0][:5])
```