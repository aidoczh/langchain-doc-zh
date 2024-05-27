# JaguarDB 矢量数据库

[JaguarDB 矢量数据库](http://www.jaguardb.com/windex.html)

1. 它是一个分布式矢量数据库

2. JaguarDB 的“ZeroMove”功能实现了即时的水平扩展性

3. 多模态：嵌入、文本、图像、视频、PDF、音频、时间序列和地理空间

4. 全主节点：允许并行读写

5. 异常检测功能

6. RAG 支持：将 LLM 与专有和实时数据结合使用

7. 共享元数据：在多个矢量索引之间共享元数据

8. 距离度量：欧氏距离、余弦距离、内积、曼哈顿距离、切比雪夫距离、汉明距离、杰卡德距离、闵可夫斯基距离

## 先决条件

在运行本文件中的示例之前，有两个要求。

1. 您必须安装并设置 JaguarDB 服务器及其 HTTP 网关服务器。

   请参考以下说明：

   [www.jaguardb.com](http://www.jaguardb.com)

2. 您必须安装 JaguarDB 的 HTTP 客户端包：

   ```
   pip install -U jaguardb-http-client
   ```

## RAG 与 Langchain

本节演示了在 langchain 软件堆栈中与 LLM 一起使用 Jaguar 进行聊天的过程。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.jaguar import Jaguar
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
""" 
将文本文件加载到一组文档中
"""
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=300)
docs = text_splitter.split_documents(documents)
"""
实例化 Jaguar 矢量存储
"""
### Jaguar HTTP 端点
url = "http://192.168.5.88:8080/fwww/"
### 使用 OpenAI 嵌入模型
embeddings = OpenAIEmbeddings()
### Pod 是矢量数据库
pod = "vdb"
### 矢量存储名称
store = "langchain_rag_store"
### 矢量索引名称
vector_index = "v"
### 矢量索引类型
# 余弦距离：距离度量
# 分数：嵌入向量是小数
# 浮点数：使用浮点数存储的值
vector_type = "cosine_fraction_float"
### 每个嵌入向量的维度
vector_dimension = 1536
### 实例化 Jaguar 存储对象
vectorstore = Jaguar(
    pod, store, vector_index, vector_type, vector_dimension, url, embeddings
)
"""
必须进行登录以授权客户端。
环境变量 JAGUAR_API_KEY 或文件 $HOME/.jagrc
应包含用于访问 JaguarDB 服务器的 API 密钥。
"""
vectorstore.login()
"""
在 JaguarDB 数据库服务器上创建矢量存储。
这应该只做一次。
"""
# 矢量存储的额外元数据字段
metadata = "category char(16)"
# 存储文本字段的字符数
text_size = 4096
# 在服务器上创建一个矢量存储
vectorstore.create(metadata, text_size)
"""
将文本拆分器中的文本添加到我们的矢量存储中
"""
vectorstore.add_documents(docs)
""" 获取检索器对象 """
retriever = vectorstore.as_retriever()
# retriever = vectorstore.as_retriever(search_kwargs={"where": "m1='123' and m2='abc'"})
""" 检索器对象可与 LangChain 和 LLM 一起使用 """
```

## 与 Jaguar 矢量存储的交互

用户可以直接与 Jaguar 矢量存储进行相似性搜索和异常检测。

```python
from langchain_community.vectorstores.jaguar import Jaguar
from langchain_openai import OpenAIEmbeddings
# 实例化 Jaguar 矢量存储对象
url = "http://192.168.3.88:8080/fwww/"
pod = "vdb"
store = "langchain_test_store"
vector_index = "v"
vector_type = "cosine_fraction_float"
vector_dimension = 10
embeddings = OpenAIEmbeddings()
vectorstore = Jaguar(
    pod, store, vector_index, vector_type, vector_dimension, url, embeddings
)
# 登录进行授权
vectorstore.login()
# 使用两个元数据字段创建矢量存储
# 这只需要运行一次。
metadata_str = "author char(32), category char(16)"
vectorstore.create(metadata_str, 1024)
# 添加文本列表
texts = ["foo", "bar", "baz"]
metadatas = [
    {"author": "Adam", "category": "Music"},
    {"author": "Eve", "category": "Music"},
    {"author": "John", "category": "History"},
]
ids = vectorstore.add_texts(texts=texts, metadatas=metadatas)
# 搜索相似文本
output = vectorstore.similarity_search(
    query="foo",
    k=1,
    metadatas=["author", "category"],
)
assert output[0].page_content == "foo"
assert output[0].metadata["author"] == "Adam"
assert output[0].metadata["category"] == "Music"
assert len(output) == 1
# 使用过滤器进行搜索（where）
where = "author='Eve'"
output = vectorstore.similarity_search(
    query="foo",
    k=3,
    fetch_k=9,
    where=where,
    metadatas=["author", "category"],
)
assert output[0].page_content == "bar"
assert output[0].metadata["author"] == "Eve"
assert output[0].metadata["category"] == "Music"
assert len(output) == 1
# 异常检测
result = vectorstore.is_anomalous(
    query="dogs can jump high",
)
assert result is False
# 清除存储中的所有数据
vectorstore.clear()
assert vectorstore.count() == 0
# 完全删除存储
vectorstore.drop()
# 注销
vectorstore.logout()
```

抱歉，我需要您提供需要翻译的英文段落才能开始工作。