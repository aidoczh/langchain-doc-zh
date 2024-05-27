# PremAI
[PremAI](https://app.premai.io) 是一个统一平台，让您能够轻松构建功能强大、可立即投入生产的 GenAI 应用程序，从而让您能够更专注于用户体验和整体增长。在本节中，我们将讨论如何通过 `PremAIEmbeddings` 来访问不同的嵌入模型。
## 安装和设置
我们首先安装 langchain 和 premai-sdk。您可以输入以下命令进行安装：
```bash
pip install premai langchain
```
在继续之前，请确保您已在 Prem 上创建了一个帐户并已启动了一个项目。如果没有，请按以下步骤免费开始：
1. 登录 [PremAI](https://app.premai.io/accounts/login/)，如果您是第一次访问，请创建您的 API 密钥 [here](https://app.premai.io/api_keys/)。
2. 进入 [app.premai.io](https://app.premai.io)，这将带您进入项目的仪表板。
3. 创建一个项目，这将生成一个项目 ID（写作 ID）。此 ID 将帮助您与部署的应用程序进行交互。
恭喜您在 Prem 上创建了您的第一个部署应用程序 🎉 现在我们可以使用 langchain 与我们的应用程序进行交互。
```python
# 让我们首先导入一些模块并定义我们的嵌入对象
from langchain_community.embeddings import PremAIEmbeddings
```
一旦我们导入了所需的模块，让我们设置客户端。现在假设我们的 `project_id` 是 8。但请确保您使用您的项目 ID，否则将会出错。
```python
import getpass
import os
if os.environ.get("PREMAI_API_KEY") is None:
    os.environ["PREMAI_API_KEY"] = getpass.getpass("PremAI API Key:")
```
```python
model = "text-embedding-3-large"
embedder = PremAIEmbeddings(project_id=8, model=model)
```
我们已经定义了我们的嵌入模型。我们支持许多嵌入模型。下面是一个表格，显示了我们支持的嵌入模型数量。
| 提供商    | Slug                                     | 上下文标记数 |
|-------------|------------------------------------------|----------------|
| cohere      | embed-english-v3.0                       | N/A            |
| openai      | text-embedding-3-small                   | 8191           |
| openai      | text-embedding-3-large                   | 8191           |
| openai      | text-embedding-ada-002                   | 8191           |
| replicate   | replicate/all-mpnet-base-v2              | N/A            |
| together    | togethercomputer/Llama-2-7B-32K-Instruct | N/A            |
| mistralai   | mistral-embed                            | 4096           |
要更改模型，您只需复制 `slug` 并访问您的嵌入模型。现在让我们开始使用我们的嵌入模型，先进行单个查询，然后进行多个查询（也称为文档）。
```python
query = "你好，这是一个测试查询"
query_result = embedder.embed_query(query)
# 让我们打印查询嵌入向量的前五个元素
print(query_result[:5])
```
```output
[-0.02129288576543331, 0.0008162345038726926, -0.004556538071483374, 0.02918623760342598, -0.02547479420900345]
```
最后让我们嵌入一个文档。
```python
documents = ["这是文档1", "这是文档2", "这是文档3"]
doc_result = embedder.embed_documents(documents)
# 与之前的结果类似，让我们打印第一个文档向量的前五个元素
print(doc_result[0][:5])
```
```output
[-0.0030691148713231087, -0.045334383845329285, -0.0161729846149683, 0.04348714277148247, -0.0036920777056366205]
```