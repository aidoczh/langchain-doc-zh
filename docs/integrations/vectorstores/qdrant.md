# Qdrant

[Qdrant](https://qdrant.tech/documentation/)（读作：quadrant）是一个向量相似度搜索引擎。它提供了一个生产就绪的服务，具有方便的 API 来存储、搜索和管理点 - 带有附加载荷的向量。`Qdrant`专门支持扩展过滤功能，使其对各种神经网络或基于语义的匹配、分面搜索和其他应用非常有用。

本笔记本展示了如何使用与`Qdrant`向量数据库相关的功能。

有各种运行`Qdrant`的模式，取决于所选择的模式，会有一些细微的差异。选项包括：

- 本地模式，无需服务器

- 本地部署服务器

- Qdrant 云

请参阅[安装说明](https://qdrant.tech/documentation/install/)。

```python
%pip install --upgrade --quiet  langchain-qdrant langchain-openai langchain
```

我们想要使用`OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```output
OpenAI API Key: ········
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_qdrant import Qdrant
from langchain_text_splitters import CharacterTextSplitter
```

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

## 从 LangChain 连接到 Qdrant

### 本地模式

Python 客户端允许您在本地模式下运行相同的代码，而无需运行 Qdrant 服务器。这对于测试和调试或者如果您计划仅存储少量向量非常有用。嵌入可能完全保存在内存中或者持久化在磁盘上。

#### 内存中

对于一些测试场景和快速实验，您可能更喜欢仅将所有数据保存在内存中，因此当客户端被销毁时数据会丢失 - 通常在脚本/笔记本的末尾。

```python
qdrant = Qdrant.from_documents(
    docs,
    embeddings,
    location=":memory:",  # 本地模式，仅内存存储
    collection_name="my_documents",
)
```

#### 磁盘存储

在不使用 Qdrant 服务器的本地模式下，还可以将您的向量存储在磁盘上，以便它们在运行之间持久化。

```python
qdrant = Qdrant.from_documents(
    docs,
    embeddings,
    path="/tmp/local_qdrant",
    collection_name="my_documents",
)
```

### 本地部署服务器

无论您选择使用 [Docker 容器](https://qdrant.tech/documentation/install/) 在本地启动 Qdrant，还是选择使用 [官方 Helm 图表](https://github.com/qdrant/qdrant-helm) 在 Kubernetes 上部署，连接到这样一个实例的方式都是相同的。您需要提供一个指向服务的 URL。

```python
url = "<---qdrant url here --->"
qdrant = Qdrant.from_documents(
    docs,
    embeddings,
    url=url,
    prefer_grpc=True,
    collection_name="my_documents",
)
```

### Qdrant 云

如果您不想忙于管理基础设施，可以选择在 [Qdrant 云](https://cloud.qdrant.io/) 上设置一个完全托管的 Qdrant 集群。包括一个永久免费的 1GB 集群供试用。使用托管版本与使用 Qdrant 的主要区别在于，您需要提供 API 密钥以防止公开访问您的部署。该值也可以设置在 `QDRANT_API_KEY` 环境变量中。

```python
url = "<---qdrant cloud cluster url here --->"
api_key = "<---api key here--->"
qdrant = Qdrant.from_documents(
    docs,
    embeddings,
    url=url,
    prefer_grpc=True,
    api_key=api_key,
    collection_name="my_documents",
)
```

## 使用现有集合

要获取一个`langchain_qdrant.Qdrant`的实例，而不加载任何新文档或文本，您可以使用`Qdrant.from_existing_collection()`方法。

```python
qdrant = Qdrant.from_existing_collection(
    embeddings=embeddings,
    collection_name="my_documents",
    url="http://localhost:6333",
)
```

## 重新创建集合

如果集合已经存在，则会重用该集合。将`force_recreate`设置为`True`允许删除旧集合并从头开始。

```python
url = "<---qdrant url here --->"
qdrant = Qdrant.from_documents(
    docs,
    embeddings,
    url=url,
    prefer_grpc=True,
    collection_name="my_documents",
    force_recreate=True,
)
```

## 相似度搜索

使用 Qdrant 向量存储的最简单场景是执行相似度搜索。在幕后，我们的查询将使用`embedding_function`对查询进行编码，并用于在 Qdrant 集合中查找相似的文档。

```python
query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search(query)
```

```python
print(found_docs[0].page_content)
```

```output
今晚，我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《揭露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生都在为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。
而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律专家之一，将继续延续布雷耶司法部长的卓越传统。
```

我知道什么是有效的：投资于犯罪预防和社区警务人员，他们会巡逻，了解社区，恢复信任和安全。

## Qdrant 作为检索器

Qdrant，像其他向量存储一样，是一个 LangChain 检索器，使用余弦相似度。

```python
retriever = qdrant.as_retriever()
```

也可以指定使用 MMR 作为搜索策略，而不是相似度。

```python
retriever = qdrant.as_retriever(search_type="mmr")
```

```python
query = "总统对Ketanji Brown Jackson说了什么"
retriever.invoke(query)[0]
```

```output
Document(page_content='今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯投票权法案》。还有，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。\n\n总统最严肃的宪法责任之一就是提名人选担任美国最高法院法官。\n\n4天前，我提名了联邦上诉法院法官Ketanji Brown Jackson。她是我们国家最顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。', metadata={'source': '../../../state_of_the_union.txt'})
```

## 定制化 Qdrant

在 Langchain 应用程序中使用现有的 Qdrant 集合有一些选项。在这种情况下，您可能需要定义如何将 Qdrant 点映射到 Langchain `Document` 中。

### 命名向量

Qdrant 支持[每个点多个向量](https://qdrant.tech/documentation/concepts/collections/#collection-with-multiple-vectors)的命名向量。Langchain 只需要每个文档一个嵌入向量，默认情况下使用单个向量。但是，如果您使用外部创建的集合或希望使用命名向量，可以通过提供其名称进行配置。

```python
Qdrant.from_documents(
    docs,
    embeddings,
    location=":memory:",
    collection_name="my_documents_2",
    vector_name="custom_vector",
)
```

作为 Langchain 用户，您不会看到使用命名向量和不使用命名向量的任何区别。Qdrant 集成会在后台处理转换。

### 元数据

Qdrant 存储您的向量嵌入以及可选的类似 JSON 的有效负载。有效负载是可选的，但由于 LangChain 假定嵌入是从文档生成的，我们保留上下文数据，这样您也可以提取原始文本。

默认情况下，您的文档将以以下有效负载结构存储：

```json
{
    "page_content": "Lorem ipsum dolor sit amet",
    "metadata": {
        "foo": "bar"
    }
}
```

但是，您可以决定使用不同的键来存储页面内容和元数据。如果您已经有一个想要重用的集合，这将非常有用。

```python
Qdrant.from_documents(
    docs,
    embeddings,
    location=":memory:",
    collection_name="my_documents_2",
    content_payload_key="my_page_content_key",
    metadata_payload_key="my_meta",
)
```