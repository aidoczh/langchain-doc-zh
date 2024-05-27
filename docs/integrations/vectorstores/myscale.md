

# MyScale

[MyScale](https://docs.myscale.com/en/overview/) 是一个基于云的数据库，专为人工智能应用和解决方案进行了优化，构建在开源项目 [ClickHouse](https://github.com/ClickHouse/ClickHouse) 的基础上。

这篇笔记展示了如何使用与 `MyScale` 向量数据库相关的功能。

## 设置环境

```python
%pip install --upgrade --quiet  clickhouse-connect
```

我们想要使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API 密钥:")
os.environ["OPENAI_API_BASE"] = getpass.getpass("OpenAI Base:")
os.environ["MYSCALE_HOST"] = getpass.getpass("MyScale 主机:")
os.environ["MYSCALE_PORT"] = getpass.getpass("MyScale 端口:")
os.environ["MYSCALE_USERNAME"] = getpass.getpass("MyScale 用户名:")
os.environ["MYSCALE_PASSWORD"] = getpass.getpass("MyScale 密码:")
```

有两种方法可以设置 myscale 索引的参数。

1. 环境变量

在运行应用程序之前，请使用 `export` 设置环境变量：

`export MYSCALE_HOST='<your-endpoints-url>' MYSCALE_PORT=<your-endpoints-port> MYSCALE_USERNAME=<your-username> MYSCALE_PASSWORD=<your-password> ...`

您可以在我们的 SaaS 上轻松找到您的账户、密码和其他信息。详情请参考[此文档](https://docs.myscale.com/en/cluster-management/)。

`MyScaleSettings` 下的每个属性都可以使用前缀 `MYSCALE_` 进行设置，且不区分大小写。

2. 使用参数创建 `MyScaleSettings` 对象

```python
from langchain_community.vectorstores import MyScale, MyScaleSettings
config = MyScaleSetting(host="<your-backend-url>", port=8443, ...)
index = MyScale(embedding_function, config)
index.add_documents(...)
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import MyScale
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
for d in docs:
    d.metadata = {"some": "metadata"}
docsearch = MyScale.from_documents(docs, embeddings)
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = docsearch.similarity_search(query)
```

```output
插入数据...: 100%|██████████| 42/42 [00:15<00:00,  2.66it/s]
```

```python
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯投票权法案》。顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统最严肃的宪法责任之一是提名某人担任美国最高法院法官。
我在4天前做到了，当时我提名了联邦上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续布雷耶法官的卓越传统。
```

## 获取连接信息和数据架构

```python
print(str(docsearch))
```

## 过滤

您可以直接访问 myscale SQL 的 where 语句。您可以按照标准 SQL 编写 `WHERE` 子句。

**注意**：请注意 SQL 注入，此接口不得由最终用户直接调用。

如果您在设置中自定义了 `column_map`，您可以像这样使用过滤器进行搜索：

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import MyScale
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
for i, d in enumerate(docs):
    d.metadata = {"doc_id": i}
docsearch = MyScale.from_documents(docs, embeddings)
```

```output
插入数据...: 100%|██████████| 42/42 [00:15<00:00,  2.68it/s]
```

### 带分数的相似性搜索

返回的距离分数是余弦距离。因此，得分越低越好。

```python
meta = docsearch.metadata_column
output = docsearch.similarity_search_with_relevance_scores(
    "总统对 Ketanji Brown Jackson 说了什么?",
    k=4,
    where_str=f"{meta}.doc_id<10",
)
for d, dist in output:
    print(dist, d.metadata, d.page_content[:20] + "...")
```

```output
0.229655921459198 {'doc_id': 0} 尊敬的议长，女士们...
0.24506962299346924 {'doc_id': 8} 还有很多家庭...
0.24786919355392456 {'doc_id': 1} 一些公民团体...
0.24875116348266602 {'doc_id': 6} 我正在采取有力...
```

## 删除您的数据

您可以使用`.drop()`方法删除表，也可以使用`.delete()`方法部分删除数据。

```python
# 直接使用`where_str`删除
docsearch.delete(where_str=f"{docsearch.metadata_column}.doc_id < 5")
meta = docsearch.metadata_column
output = docsearch.similarity_search_with_relevance_scores(
    "总统对Ketanji Brown Jackson有什么看法？",
    k=4,
    where_str=f"{meta}.doc_id<10",
)
for d, dist in output:
    print(dist, d.metadata, d.page_content[:20] + "...")
```

```output
0.24506962299346924 {'doc_id': 8} 还有很多家庭...
0.24875116348266602 {'doc_id': 6} 我正在采取有力...
0.26027143001556396 {'doc_id': 7} 我们看到了团结...
0.26390212774276733 {'doc_id': 9} 不像之前的2万亿美...
```

```python
docsearch.drop()
```