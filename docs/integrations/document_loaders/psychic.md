# 心灵感应

本文介绍如何从 `Psychic` 加载文档。更多详情请参见[这里](/docs/integrations/providers/psychic)。

## 先决条件

1. 按照[此文档](/docs/integrations/providers/psychic)中的快速入门部分进行操作。

2. 登录[Psychic仪表板](https://dashboard.psychic.dev/)并获取您的密钥。

3. 在您的Web应用程序中安装前端React库，并让用户进行身份验证连接。连接将使用您指定的连接ID创建。

## 加载文档

使用 `PsychicLoader` 类从连接中加载文档。每个连接都有一个连接器ID（对应已连接的SaaS应用程序）和一个连接ID（您传递给前端库的）。

```python
# 如果您尚未安装psychicapi，请取消下面的注释以安装
!poetry run pip -q install psychicapi langchain-chroma
```

```output
[notice] A new release of pip is available: 23.0.1 -> 23.1.2
[notice] To update, run: pip install --upgrade pip
```

```python
from langchain_community.document_loaders import PsychicLoader
from psychicapi import ConnectorId
# 创建一个用于Google Drive的文档加载器。我们也可以通过将connector_id设置为适当的值（例如ConnectorId.notion.value）来从其他连接器加载。
# 此加载器使用我们的测试凭据
google_drive_loader = PsychicLoader(
    api_key="7ddb61c1-8b6a-4d31-a58e-30d1c9ea480e",
    connector_id=ConnectorId.gdrive.value,
    connection_id="google-test",
)
documents = google_drive_loader.load()
```

## 将文档转换为嵌入向量

现在，我们可以将这些文档转换为嵌入向量，并将它们存储在像Chroma这样的向量数据库中。

```python
from langchain.chains import RetrievalQAWithSourcesChain
from langchain_chroma import Chroma
from langchain_openai import OpenAI, OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
docsearch = Chroma.from_documents(texts, embeddings)
chain = RetrievalQAWithSourcesChain.from_chain_type(
    OpenAI(temperature=0), chain_type="stuff", retriever=docsearch.as_retriever()
)
chain({"question": "什么是心灵感应？"}, return_only_outputs=True)
```