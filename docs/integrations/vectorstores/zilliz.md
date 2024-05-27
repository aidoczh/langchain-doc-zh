# Zilliz

>[Zilliz Cloud](https://zilliz.com/doc/quick_start) 是一个完全托管在云端的 `LF AI Milvus®` 服务，

这个笔记本展示了如何使用与 Zilliz Cloud 托管的向量数据库相关的功能。

在运行之前，您应该已经启动并运行了一个 `Zilliz Cloud` 实例。这里是[安装说明](https://zilliz.com/cloud)。

```python
%pip install --upgrade --quiet  pymilvus
```

我们想要使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API 密钥:")
```

```output
OpenAI API 密钥:········
```

```python
# 替换
ZILLIZ_CLOUD_URI = ""  # 示例: "https://in01-17f69c292d4a5sa.aws-us-west-2.vectordb.zillizcloud.com:19536"
ZILLIZ_CLOUD_USERNAME = ""  # 示例: "username"
ZILLIZ_CLOUD_PASSWORD = ""  # 示例: "*********"
ZILLIZ_CLOUD_API_KEY = ""  # 示例: "*********" (用于无服务器集群，可用作用户和密码的替代)
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Milvus
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

```python
vector_db = Milvus.from_documents(
    docs,
    embeddings,
    connection_args={
        "uri": ZILLIZ_CLOUD_URI,
        "user": ZILLIZ_CLOUD_USERNAME,
        "password": ZILLIZ_CLOUD_PASSWORD,
        # "token": ZILLIZ_CLOUD_API_KEY,  # API 密钥，用于无服务器集群，可用作用户和密码的替代
        "secure": True,
    },
)
```

```python
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = vector_db.similarity_search(query)
```

```python
docs[0].page_content
```

```output
'今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法案》。而且，顺便说一下，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个致力于为这个国家服务的人表示敬意：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者和即将退休的美国最高法院法官。布雷耶法官，感谢您的服务。\n\n总统最重要的宪法责任之一就是提名人选担任美国最高法院法官。\n\n而我在4天前就已经这样做了，当时我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家最顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。'
```