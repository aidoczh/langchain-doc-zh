# ChatGPT 插件

[OpenAI 插件](https://platform.openai.com/docs/plugins/introduction) 将 `ChatGPT` 与第三方应用程序连接起来。这些插件使 `ChatGPT` 能够与开发人员定义的 API 进行交互，增强了 `ChatGPT` 的功能，并使其能够执行各种操作。

插件使 `ChatGPT` 能够执行以下操作：

- 检索实时信息；例如体育比分、股票价格、最新新闻等。

- 检索知识库信息；例如公司文档、个人笔记等。

- 代表用户执行操作；例如订机票、订餐等。

这个笔记本展示了如何在 LangChain 中使用 ChatGPT Retriever 插件。

```python
# 步骤 1：加载
# 使用 LangChain 的 DocumentLoaders 加载文档
# 参考 https://langchain.readthedocs.io/en/latest/modules/document_loaders/examples/csv.html
from langchain_community.document_loaders import CSVLoader
from langchain_core.documents import Document
loader = CSVLoader(
    file_path="../../document_loaders/examples/example_data/mlb_teams_2012.csv"
)
data = loader.load()
# 步骤 2：转换
# 将文档转换为 https://github.com/openai/chatgpt-retrieval-plugin 期望的格式
import json
from typing import List
def write_json(path: str, documents: List[Document]) -> None:
    results = [{"text": doc.page_content} for doc in documents]
    with open(path, "w") as f:
        json.dump(results, f, indent=2)
write_json("foo.json", data)
# 步骤 3：使用
# 像处理其他 json 文件一样处理这个文件 https://github.com/openai/chatgpt-retrieval-plugin/tree/main/scripts/process_json
```

## 使用 ChatGPT Retriever 插件

好的，我们已经创建了 ChatGPT Retriever 插件，但我们该如何使用它呢？

下面的代码演示了如何使用它。

我们想要使用 `ChatGPTPluginRetriever`，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```output
OpenAI API Key: ········
```

```python
from langchain_community.retrievers import (
    ChatGPTPluginRetriever,
)
```

```python
retriever = ChatGPTPluginRetriever(url="http://0.0.0.0:8000", bearer_token="foo")
```

```python
retriever.invoke("alice's phone number")
```

```output
[Document(page_content="This is Alice's phone number: 123-456-7890", lookup_str='', metadata={'id': '456_0', 'metadata': {'source': 'email', 'source_id': '567', 'url': None, 'created_at': '1609592400.0', 'author': 'Alice', 'document_id': '456'}, 'embedding': None, 'score': 0.925571561}, lookup_index=0),
 Document(page_content='This is a document about something', lookup_str='', metadata={'id': '123_0', 'metadata': {'source': 'file', 'source_id': 'https://example.com/doc1', 'url': 'https://example.com/doc1', 'created_at': '1609502400.0', 'author': 'Alice', 'document_id': '123'}, 'embedding': None, 'score': 0.6987589}, lookup_index=0),
 Document(page_content='Team: Angels "Payroll (millions)": 154.49 "Wins": 89', lookup_str='', metadata={'id': '59c2c0c1-ae3f-4272-a1da-f44a723ea631_0', 'metadata': {'source': None, 'source_id': None, 'url': None, 'created_at': None, 'author': None, 'document_id': '59c2c0c1-ae3f-4272-a1da-f44a723ea631'}, 'embedding': None, 'score': 0.697888613}, lookup_index=0)]
```