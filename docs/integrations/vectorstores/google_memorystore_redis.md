# Google Memorystore for Redis

[Google Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/memorystore-for-redis-overview) 是一项由 Redis 内存数据存储支持的全托管服务，用于构建应用程序缓存，提供亚毫秒级的数据访问。通过使用 Memorystore for Redis 的 Langchain 集成，可以扩展数据库应用程序以构建基于人工智能的体验。

本笔记将介绍如何使用 [Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/memorystore-for-redis-overview) 存储向量嵌入，使用 `MemorystoreVectorStore` 类。

在 [GitHub](https://github.com/googleapis/langchain-google-memorystore-redis-python/) 上了解有关该软件包的更多信息。

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-memorystore-redis-python/blob/main/docs/vector_store.ipynb)

## 先决条件

### 开始之前

要运行此笔记，您需要执行以下操作：

- [创建 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

- [启用 Memorystore for Redis API](https://console.cloud.google.com/flows/enableapi?apiid=redis.googleapis.com)

- [创建 Memorystore for Redis 实例](https://cloud.google.com/memorystore/docs/redis/create-instance-console)。确保版本大于或等于 7.2。

### 🦜🔗 库安装

集成位于其自己的 `langchain-google-memorystore-redis` 软件包中，因此我们需要安装它。

```python
%pip install -upgrade --quiet langchain-google-memorystore-redis langchain
```

**仅适用于 Colab：**取消注释以下单元格以重新启动内核，或使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 自动安装后重新启动内核，以便您的环境可以访问新的软件包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在此笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

- 运行 `gcloud config list`。

- 运行 `gcloud projects list`。

- 参见支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填写您的 Google Cloud 项目 ID，然后运行单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 🔐 身份验证

作为在此笔记本中登录的 IAM 用户，进行 Google Cloud 身份验证，以访问您的 Google Cloud 项目。

- 如果您使用 Colab 运行此笔记，使用下面的单元格并继续。

- 如果您使用 Vertex AI Workbench，请查看[此处](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)的设置说明。

```python
from google.colab import auth
auth.authenticate_user()
```

## 基本用法

### 初始化向量索引

```python
import redis
from langchain_google_memorystore_redis import (
    DistanceStrategy,
    HNSWConfig,
    RedisVectorStore,
)
# 连接到 Memorystore for Redis 实例
redis_client = redis.from_url("redis://127.0.0.1:6379")
# 使用描述性参数配置 HNSW 索引
index_config = HNSWConfig(
    name="my_vector_index", distance_strategy=DistanceStrategy.COSINE, vector_size=128
)
# 初始化/创建向量存储索引
RedisVectorStore.init_index(client=redis_client, index_config=index_config)
```

### 准备文档

在与向量存储交互之前，文本需要处理和数值表示。这涉及以下操作：

- 加载文本：TextLoader 从文件中获取文本数据（例如 "state_of_the_union.txt"）。

- 文本拆分：CharacterTextSplitter 将文本分成较小的块，以供嵌入模型使用。

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("./state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

### 将文档添加到向量存储

在文本准备和嵌入生成之后，以下方法将它们插入到 Redis 向量存储中。

#### 方法 1：用于直接插入的类方法

此方法结合了嵌入创建和插入为一步，使用 from_documents 类方法：

```python
from langchain_community.embeddings.fake import FakeEmbeddings
embeddings = FakeEmbeddings(size=128)
redis_client = redis.from_url("redis://127.0.0.1:6379")
rvs = RedisVectorStore.from_documents(
    docs, embedding=embeddings, client=redis_client, index_name="my_vector_index"
)
```

#### 方法二：基于实例的插入

这种方法在使用新的或现有的 RedisVectorStore 时提供了灵活性：

- [可选] 创建 RedisVectorStore 实例：实例化一个 RedisVectorStore 对象以进行自定义。如果您已经有一个实例，请继续下一步。

- 添加带有元数据的文本：将原始文本和元数据提供给实例。嵌入生成和插入到向量存储中将自动处理。

```python
rvs = RedisVectorStore(
    client=redis_client, index_name="my_vector_index", embeddings=embeddings
)
ids = rvs.add_texts(
    texts=[d.page_content for d in docs], metadatas=[d.metadata for d in docs]
)
```

### 执行相似性搜索（KNN）

向量存储填充后，可以搜索与查询语义上相似的文本。以下是如何使用默认设置的 KNN（K-最近邻）：

- 构建查询：自然语言问题表达了搜索意图（例如，“总统对 Ketanji Brown Jackson 说了什么”）。

- 检索相似结果：`similarity_search` 方法找到向量存储中与查询意义最接近的项目。

```python
import pprint
query = "总统对 Ketanji Brown Jackson 说了什么"
knn_results = rvs.similarity_search(query=query)
pprint.pprint(knn_results)
```

### 执行基于范围的相似性搜索

范围查询通过指定所需的相似性阈值和查询文本提供了更多的控制：

- 构建查询：自然语言问题定义了搜索意图。

- 设置相似性阈值：distance_threshold 参数确定了必须被视为相关的匹配有多接近。

- 检索结果：`similarity_search_with_score` 方法找到向量存储中在指定相似性阈值范围内的项目。

```python
rq_results = rvs.similarity_search_with_score(query=query, distance_threshold=0.8)
pprint.pprint(rq_results)
```

### 执行最大边际相关性（MMR）搜索

MMR 查询旨在找到既与查询相关又彼此不同的结果，减少搜索结果中的冗余。

- 构建查询：自然语言问题定义了搜索意图。

- 平衡相关性和多样性：lambda_mult 参数控制了严格相关性和促进结果多样性之间的权衡。

- 检索 MMR 结果：`max_marginal_relevance_search` 方法根据 lambda 设置返回优化相关性和多样性组合的项目。

```python
mmr_results = rvs.max_marginal_relevance_search(query=query, lambda_mult=0.90)
pprint.pprint(mmr_results)
```

## 将向量存储用作检索器

为了与其他 LangChain 组件无缝集成，可以将向量存储转换为检索器。这提供了几个优势：

- LangChain 兼容性：许多 LangChain 工具和方法都设计为直接与检索器交互。

- 使用简便：`as_retriever()` 方法将向量存储转换为简化查询的格式。

```python
retriever = rvs.as_retriever()
results = retriever.invoke(query)
pprint.pprint(results)
```

## 清理

### 从向量存储中删除文档

偶尔需要从向量存储中删除文档（及其关联的向量）。`delete` 方法提供了这个功能。

```python
rvs.delete(ids)
```

### 删除向量索引

可能有一些情况需要删除现有的向量索引。常见的原因包括：

- 索引配置更改：如果需要修改索引参数，则通常需要删除并重新创建索引。

- 存储管理：删除未使用的索引可以帮助释放 Redis 实例中的空间。

注意：向量索引的删除是一个不可逆的操作。在继续之前，请确保不再需要存储的向量和搜索功能。

```python
# 删除向量索引
RedisVectorStore.drop_index(client=redis_client, index_name="my_vector_index")
```

