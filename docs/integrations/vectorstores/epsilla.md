# Epsilla

[Epsilla](https://www.epsilla.com) 是一个开源的矢量数据库，利用先进的并行图遍历技术进行矢量索引。Epsilla 使用 GPL-3.0 许可证。

本笔记展示了如何使用与 `Epsilla` 矢量数据库相关的功能。

作为先决条件，您需要运行 Epsilla 矢量数据库（例如，通过我们的 Docker 镜像），并安装 ``pyepsilla`` 包。在 [文档](https://epsilla-inc.gitbook.io/epsilladb/quick-start) 中查看完整文档。

```python
!pip/pip3 install pyepsilla
```

我们想要使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

OpenAI API 密钥: ········

```python
from langchain_community.vectorstores import Epsilla
from langchain_openai import OpenAIEmbeddings
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
documents = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0).split_documents(
    documents
)
embeddings = OpenAIEmbeddings()
```

Epsilla 矢量数据库正在使用默认主机 "localhost" 和端口 "8888" 运行。我们有自定义的数据库路径、数据库名称和集合名称，而不是默认的。

```python
from pyepsilla import vectordb
client = vectordb.Client()
vector_store = Epsilla.from_documents(
    documents,
    embeddings,
    client,
    db_path="/tmp/mypath",
    db_name="MyDB",
    collection_name="MyCollection",
)
```

```python
query = "总统对Ketanji Brown Jackson有什么看法"
docs = vector_store.similarity_search(query)
print(docs[0].page_content)
```

在一个又一个州，新法律不仅通过了压制选举的投票，而且颠覆了整个选举。

我们不能让这种事情发生。

今晚，我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法案》。而且，顺便说一句，通过《揭露法案》，这样美国人就可以知道谁资助了我们的选举。

今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。

总统担负的最严肃的宪法责任之一就是提名某人担任美国最高法院的法官。

而我在4天前就做到了，当我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。