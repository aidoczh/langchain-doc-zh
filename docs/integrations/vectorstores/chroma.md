# Chroma

[Chroma](https://docs.trychroma.com/getting-started) 是一个以人工智能为基础的开源向量数据库，专注于开发者的生产力和幸福感。Chroma 使用 Apache 2.0 许可证。

使用以下命令安装 Chroma：

```sh
pip install langchain-chroma
```

Chroma 可以以多种模式运行。以下是每种模式的示例，均与 LangChain 集成：

- `in-memory` - 在 Python 脚本或 Jupyter 笔记本中

- `in-memory with persistance` - 在脚本或笔记本中保存/加载到磁盘

- `in a docker container` - 作为在本地机器或云中运行的服务器

与任何其他数据库一样，您可以进行以下操作：

- `.add`

- `.get`

- `.update`

- `.upsert`

- `.delete`

- `.peek`

- 而 `.query` 则运行相似性搜索。

查看完整文档，请访问 [docs](https://docs.trychroma.com/reference/Collection)。要直接访问这些方法，可以使用 `._collection.method()`。

## 基本示例

在这个基本示例中，我们获取最近的国情咨文，将其分割成片段，使用开源嵌入模型进行嵌入，加载到 Chroma 中，然后进行查询。

```python
# 导入
from langchain_chroma import Chroma
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.sentence_transformer import SentenceTransformerEmbeddings
from langchain_text_splitters import CharacterTextSplitter
# 加载文档并将其分割成片段
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
# 将其分割成片段
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
# 创建开源嵌入函数
embedding_function = SentenceTransformerEmbeddings(model_name="all-MiniLM-L6-v2")
# 将其加载到 Chroma 中
db = Chroma.from_documents(docs, embedding_function)
# 进行查询
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
# 打印结果
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法》。而且，顺便说一句，通过《披露法》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统最严肃的宪法责任之一就是提名某人担任美国最高法院的法官。
而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智囊，将继续延续布雷耶法官的卓越传统。
```

## 基本示例（包括保存到磁盘）

在上一个示例的基础上，如果您想要保存到磁盘，只需初始化 Chroma 客户端并传递要保存数据的目录。

`注意`：Chroma 尽最大努力自动将数据保存到磁盘，但多个内存客户端可能会相互干扰。最佳做法是，任何给定时间只运行一个客户端。

```python
# 保存到磁盘
db2 = Chroma.from_documents(docs, embedding_function, persist_directory="./chroma_db")
docs = db2.similarity_search(query)
# 从磁盘加载
db3 = Chroma(persist_directory="./chroma_db", embedding_function=embedding_function)
docs = db3.similarity_search(query)
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法》。而且，顺便说一句，通过《披露法》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统最严肃的宪法责任之一就是提名某人担任美国最高法院的法官。
而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智囊，将继续延续布雷耶法官的卓越传统。
```

## 将 Chroma 客户端传递给 Langchain

您还可以创建一个 Chroma 客户端并将其传递给 LangChain。如果您希望更轻松地访问底层数据库，这将特别有用。

您还可以指定要让 LangChain 使用的集合名称。

```python
import chromadb
persistent_client = chromadb.PersistentClient()
collection = persistent_client.get_or_create_collection("collection_name")
collection.add(ids=["1", "2", "3"], documents=["a", "b", "c"])
langchain_chroma = Chroma(
    client=persistent_client,
    collection_name="collection_name",
    embedding_function=embedding_function,
)
print("在集合中有", langchain_chroma._collection.count(), "个文档")
```

```output
现有嵌入 ID 的添加：1
现有嵌入 ID 的添加：2
现有嵌入 ID 的添加：3
现有嵌入 ID 的添加：1
现有嵌入 ID 的添加：2
现有嵌入 ID 的添加：3
现有嵌入 ID 的添加：1
现有嵌入 ID 的插入：1
现有嵌入 ID 的添加：2
现有嵌入 ID 的插入：2
现有嵌入 ID 的添加：3
现有嵌入 ID 的插入：3
```

在集合中有 3 个项目

## 基本示例（使用 Docker 容器）

您还可以在 Docker 容器中单独运行 Chroma 服务器，创建一个客户端连接到它，然后将其传递给 LangChain。

Chroma 能够处理多个文档的“集合”，但 LangChain 接口只期望一个集合，因此我们需要指定集合名称。LangChain 默认使用的集合名称是 "langchain"。

以下是如何克隆、构建和运行 Docker 镜像的步骤：

```sh
git clone git@github.com:chroma-core/chroma.git
```

编辑 `docker-compose.yml` 文件，在 `environment` 下添加 `ALLOW_RESET=TRUE`

```yaml
    ...
    command: uvicorn chromadb.app:app --reload --workers 1 --host 0.0.0.0 --port 8000 --log-config log_config.yml
    environment:
      - IS_PERSISTENT=TRUE
      - ALLOW_RESET=TRUE
    ports:
      - 8000:8000
    ...
```

然后运行 `docker-compose up -d --build`

```python
# 创建 Chroma 客户端
import uuid
import chromadb
from chromadb.config import Settings
client = chromadb.HttpClient(settings=Settings(allow_reset=True))
client.reset()  # 重置数据库
collection = client.create_collection("my_collection")
for doc in docs:
    collection.add(
        ids=[str(uuid.uuid1())], metadatas=doc.metadata, documents=doc.page_content
    )
# 告诉 LangChain 使用我们的客户端和集合名称
db4 = Chroma(
    client=client,
    collection_name="my_collection",
    embedding_function=embedding_function,
)
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = db4.similarity_search(query)
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法》。还有，通过《揭示法案》，这样美国人就可以知道谁资助了我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一是提名某人担任美国最高法院法官。
4 天前，我提名了联邦上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续布雷耶法官的卓越传统。
```

## 更新和删除

在构建真实应用程序时，您不仅希望添加数据，还希望更新和删除数据。

Chroma 要求用户提供 `ids` 来简化这里的簿记工作。`ids` 可以是文件名，也可以是类似 `filename_paragraphNumber` 的组合哈希值。

Chroma 支持所有这些操作，尽管有些操作仍在通过 LangChain 接口进行整合。额外的工作流改进将很快添加。

以下是一个基本示例，展示如何执行各种操作：

```python
# 创建简单的 ids
ids = [str(i) for i in range(1, len(docs) + 1)]
# 添加数据
example_db = Chroma.from_documents(docs, embedding_function, ids=ids)
docs = example_db.similarity_search(query)
print(docs[0].metadata)
# 更新文档的元数据
docs[0].metadata = {
    "source": "../../how_to/state_of_the_union.txt",
    "new_value": "hello world",
}
example_db.update_document(ids[0], docs[0])
print(example_db._collection.get(ids=[ids[0]]))
# 删除最后一个文档
print("删除前计数", example_db._collection.count())
example_db._collection.delete(ids=[ids[-1]])
print("删除后计数", example_db._collection.count())
```

```output
{'source': '../../../state_of_the_union.txt'}
```

```markdown
今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《揭示法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个终生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一是提名某人担任美国最高法院大法官。
而我在4天前就做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智囊之一，将继续延续布雷耶司法部长的卓越传统。
```

count before 46

count after 45

## 使用 OpenAI Embeddings

许多人喜欢使用 OpenAIEmbeddings，以下是如何设置它。

```python
# 获取令牌：https://platform.openai.com/account/api-keys
from getpass import getpass
from langchain_openai import OpenAIEmbeddings
OPENAI_API_KEY = getpass()
```

```python
import os
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

```python
embeddings = OpenAIEmbeddings()
new_client = chromadb.EphemeralClient()
openai_lc_client = Chroma.from_documents(
    docs, embeddings, client=new_client, collection_name="openai_collection"
)
query = "总统对凯坦吉·布朗·杰克逊说了什么"
docs = openai_lc_client.similarity_search(query)
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《揭示法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个终生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一是提名某人担任美国最高法院大法官。
而我在4天前就做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智囊之一，将继续延续布雷耶司法部长的卓越传统。
```

***

## 其他信息

### 带分数的相似性搜索

返回的距离分数是余弦距离。因此，分数越低越好。

```python
docs = db.similarity_search_with_score(query)
```

```python
docs[0]
```

```output
(Document(page_content='今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《揭示法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个终生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶司法部长，感谢您的服务。\n\n总统最严肃的宪法责任之一是提名某人担任美国最高法院大法官。\n\n而我在4天前就做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智囊之一，将继续延续布雷耶司法部长的卓越传统。', metadata={'source': '../../../state_of_the_union.txt'}),
 1.1972057819366455)
```

### 检索器选项

本节介绍如何在检索器中使用 Chroma 的不同选项。

#### MMR

除了在检索器对象中使用相似性搜索外，您还可以使用 `mmr`。

```python
retriever = db.as_retriever(search_type="mmr")
```

```python
retriever.invoke(query)[0]
```

```output
Document(page_content='今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《揭示法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个终生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶司法部长，感谢您的服务。\n\n总统最严肃的宪法责任之一是提名某人担任美国最高法院大法官。\n\n而我在4天前就做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智囊之一，将继续延续布雷耶司法部长的卓越传统。', metadata={'source': '../../../state_of_the_union.txt'})
```

### 在元数据上进行过滤

在处理之前，将集合缩小范围可能会有所帮助。

例如，可以使用 get 方法根据元数据对集合进行过滤。

```python
# 筛选已更新来源的集合
example_db.get(where={"source": "some_other_source"})
```

```output
{'ids': [], 'embeddings': None, 'metadatas': [], 'documents': []}
```