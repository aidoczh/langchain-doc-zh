# Rockset

[Rockset](https://rockset.com/) 是一个专为云端构建的实时搜索和分析数据库。Rockset 使用 [Converged Index™](https://rockset.com/blog/converged-indexing-the-secret-sauce-behind-rocksets-fast-queries/) 和高效的向量嵌入存储，以提供低延迟、高并发的大规模搜索查询。Rockset 全面支持元数据过滤，并处理不断更新的实时摄入流数据。

本笔记演示了如何在 LangChain 中将 `Rockset` 用作向量存储。在开始之前，请确保您可以访问 `Rockset` 帐户并且有可用的 API 密钥。 [立即开始免费试用。](https://rockset.com/create/)

## 设置您的环境

1. 利用 `Rockset` 控制台创建一个 [collection](https://rockset.com/docs/collections/)，并将写 API 作为数据源。在本演示中，我们创建了一个名为 `langchain_demo` 的 collection。

    配置以下 [摄入转换](https://rockset.com/docs/ingest-transformation/) 来标记您的嵌入字段，并利用性能和存储优化：

    (我们在这个示例中使用了 OpenAI `text-embedding-ada-002`，其中 #length_of_vector_embedding = 1536)

    ```
    SELECT _input.* EXCEPT(_meta), 
    VECTOR_ENFORCE(_input.description_embedding, #length_of_vector_embedding, 'float') as description_embedding 
    FROM _input
    ```

2. 创建集合后，使用控制台检索 [API 密钥](https://rockset.com/docs/iam/#users-api-keys-and-roles)。在本笔记中，我们假设您正在使用 `Oregon(us-west-2)` 区域。

3. 安装 [rockset-python-client](https://github.com/rockset/rockset-python-client) 以使 LangChain 能够直接与 `Rockset` 通信。

    ```python
    %pip install --upgrade --quiet  rockset
    ```

## LangChain 教程

在您自己的 Python 笔记本中跟着进行，生成并存储 Rockset 中的向量嵌入。

开始使用 Rockset 搜索与您的搜索查询相似的文档。

### 1. 定义关键变量

```python
import os
import rockset
ROCKSET_API_KEY = os.environ.get(
    "ROCKSET_API_KEY"
)  # 验证 ROCKSET_API_KEY 环境变量
ROCKSET_API_SERVER = rockset.Regions.usw2a1  # 验证 Rockset 区域
rockset_client = rockset.RocksetClient(ROCKSET_API_SERVER, ROCKSET_API_KEY)
COLLECTION_NAME = "langchain_demo"
TEXT_KEY = "description"
EMBEDDING_KEY = "description_embedding"
```

### 2. 准备文档

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Rockset
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

### 3. 插入文档

```python
embeddings = OpenAIEmbeddings()  # 验证 OPENAI_API_KEY 环境变量
docsearch = Rockset(
    client=rockset_client,
    embeddings=embeddings,
    collection_name=COLLECTION_NAME,
    text_key=TEXT_KEY,
    embedding_key=EMBEDDING_KEY,
)
ids = docsearch.add_texts(
    texts=[d.page_content for d in docs],
    metadatas=[d.metadata for d in docs],
)
```

### 4. 搜索相似文档

```python
query = "What did the president say about Ketanji Brown Jackson"
output = docsearch.similarity_search_with_relevance_scores(
    query, 4, Rockset.DistanceFunction.COSINE_SIM
)
print("output length:", len(output))
for d, dist in output:
    print(dist, d.metadata, d.page_content[:20] + "...")
```

### 5. 使用过滤搜索相似文档

```python
output = docsearch.similarity_search_with_relevance_scores(
    query,
    4,
    Rockset.DistanceFunction.COSINE_SIM,
    where_str="{} NOT LIKE '%citizens%'".format(TEXT_KEY),
)
print("output length:", len(output))
for d, dist in output:
    print(dist, d.metadata, d.page_content[:20] + "...")
```

### 6. [可选] 删除已插入的文档

你必须拥有与每个文档相关联的唯一ID，才能从你的集合中删除它们。

在使用 `Rockset.add_texts()` 插入文档时定义ID。否则，Rockset会为每个文档生成一个唯一ID。无论如何，`Rockset.add_texts()` 都会返回已插入文档的ID。

要删除这些文档，只需使用 `Rockset.delete_texts()` 函数。

```python
docsearch.delete_texts(ids)
```

## 摘要

在本教程中，我们成功创建了一个 `Rockset` 集合，用 OpenAI 嵌入插入了文档，并且使用了有和没有元数据过滤器的方式搜索了相似的文档。

请关注 https://rockset.com/ 以获取此领域的未来更新。