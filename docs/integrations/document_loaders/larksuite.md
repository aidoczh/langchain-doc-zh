# LarkSuite（飞书）

>[LarkSuite](https://www.larksuite.com/) 是字节跳动开发的企业协作平台。

本笔记介绍如何从 `LarkSuite` 的 REST API 中加载数据，以便将其转换为 LangChain 可以接受的格式，并提供文本摘要的示例用法。

LarkSuite API 需要访问令牌（tenant_access_token 或 user_access_token），请查看 [LarkSuite 开放平台文档](https://open.larksuite.com/document) 以获取 API 详细信息。

```python
from getpass import getpass
from langchain_community.document_loaders.larksuite import (
    LarkSuiteDocLoader,
    LarkSuiteWikiLoader,
)
DOMAIN = input("larksuite domain")
ACCESS_TOKEN = getpass("larksuite tenant_access_token or user_access_token")
DOCUMENT_ID = input("larksuite document id")
```

## 从文档加载

```python
from pprint import pprint
larksuite_loader = LarkSuiteDocLoader(DOMAIN, ACCESS_TOKEN, DOCUMENT_ID)
docs = larksuite_loader.load()
pprint(docs)
```

```output
[Document(page_content='Test Doc\nThis is a Test Doc\n\n1\n2\n3\n\n', metadata={'document_id': 'V76kdbd2HoBbYJxdiNNccajunPf', 'revision_id': 11, 'title': 'Test Doc'})]
```

## 从 Wiki 加载

```python
from pprint import pprint
DOCUMENT_ID = input("larksuite wiki id")
larksuite_loader = LarkSuiteWikiLoader(DOMAIN, ACCESS_TOKEN, DOCUMENT_ID)
docs = larksuite_loader.load()
pprint(docs)
```

```output
[Document(page_content='Test doc\nThis is a test wiki doc.\n', metadata={'document_id': 'TxOKdtMWaoSTDLxYS4ZcdEI7nwc', 'revision_id': 15, 'title': 'Test doc'})]
```

```python
# 有关更多详细信息，请参阅 https://python.langchain.com/docs/use_cases/summarization
from langchain.chains.summarize import load_summarize_chain
from langchain_community.llms.fake import FakeListLLM
llm = FakeListLLM()
chain = load_summarize_chain(llm, chain_type="map_reduce")
chain.run(docs)
```