# scikit-learn

[scikit-learn](https://scikit-learn.org/stable/) 是一个开源的机器学习算法集合，其中包括一些 [k 最近邻居](https://scikit-learn.org/stable/modules/generated/sklearn.neighbors.NearestNeighbors.html) 的实现。`SKLearnVectorStore` 封装了这个实现，并添加了将向量存储为 json、bson（二进制 json）或 Apache Parquet 格式的功能。

本文档展示了如何使用 `SKLearnVectorStore` 向量数据库。

```python
%pip install --upgrade --quiet  scikit-learn
# # 如果您计划使用 bson 序列化，请同时安装：
%pip install --upgrade --quiet  bson
# # 如果您计划使用 parquet 序列化，请同时安装：
%pip install --upgrade --quiet  pandas pyarrow
```

要使用 OpenAI 嵌入，您将需要一个 OpenAI 密钥。您可以在 https://platform.openai.com/account/api-keys 获取一个，或者随意使用其他嵌入。

```python
import os
from getpass import getpass
os.environ["OPENAI_API_KEY"] = getpass("输入您的 OpenAI 密钥:")
```

## 基本用法

### 加载示例文档语料库

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import SKLearnVectorStore
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

### 创建 SKLearnVectorStore，索引文档语料库并运行示例查询

```python
import tempfile
persist_path = os.path.join(tempfile.gettempdir(), "union.parquet")
vector_store = SKLearnVectorStore.from_documents(
    documents=docs,
    embedding=embeddings,
    persist_path=persist_path,  # persist_path 和 serializer 是可选的
    serializer="parquet",
)
query = "总统对 Ketanji Brown Jackson 有何看法"
docs = vector_store.similarity_search(query)
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。而且，顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统拥有的最严肃的宪法责任之一是提名某人担任美国最高法院法官。
而我在 4 天前就做到了，当时我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律智囊之一，将继续延续布雷耶法官的卓越传统。
```

## 保存和加载向量存储

```python
vector_store.persist()
print("向量存储已保存至", persist_path)
```

```output
向量存储已保存至 /var/folders/6r/wc15p6m13nl_nl_n_xfqpc5c0000gp/T/union.parquet
```

```python
vector_store2 = SKLearnVectorStore(
    embedding=embeddings, persist_path=persist_path, serializer="parquet"
)
print("从", persist_path, "加载了一个新的向量存储实例")
```

```output
从 /var/folders/6r/wc15p6m13nl_nl_n_xfqpc5c0000gp/T/union.parquet 加载了一个新的向量存储实例
```

```python
docs = vector_store2.similarity_search(query)
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。而且，顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统拥有的最严肃的宪法责任之一是提名某人担任美国最高法院法官。
而我在 4 天前就做到了，当时我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律智囊之一，将继续延续布雷耶法官的卓越传统。
```

## 清理

```python
os.remove(persist_path)
```