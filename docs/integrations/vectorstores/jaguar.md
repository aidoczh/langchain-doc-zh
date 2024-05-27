# Jaguar Vector Database

1. Jaguar Vector Database 是一个分布式向量数据库。

2. JaguarDB 的 "ZeroMove" 功能可以实现即时的水平扩展性。

3. 多模态：支持嵌入向量、文本、图像、视频、PDF、音频、时间序列和地理空间数据。

4. 全主节点：支持并行读写操作。

5. 异常检测功能。

6. RAG 支持：将 LLM（Large Language Model）与专有和实时数据相结合。

7. 共享元数据：可以在多个向量索引之间共享元数据。

8. 距离度量：欧氏距离、余弦距离、内积、曼哈顿距离、切比雪夫距离、汉明距离、杰卡德距离、闵可夫斯基距离。

## 先决条件

在运行本文件中的示例之前，需要满足以下两个要求：

1. 必须安装并设置 JaguarDB 服务器及其 HTTP 网关服务器。

   请参考以下说明：

   [www.jaguardb.com](http://www.jaguardb.com)

   在 Docker 环境中快速设置：

   docker pull jaguardb/jaguardb_with_http

   docker run -d -p 8888:8888 -p 8080:8080 --name jaguardb_with_http  jaguardb/jaguardb_with_http

2. 必须安装 JaguarDB 的 HTTP 客户端包：

   ```
   pip install -U jaguardb-http-client
   ```

## 使用 Langchain 进行 RAG

本节演示了在 langchain 软件栈中与 LLM（Large Language Model）和 Jaguar 一起进行聊天的过程。

```python
from langchain.chains import RetrievalQAWithSourcesChain
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.jaguar import Jaguar
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI, OpenAI, OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
""" 
将文本文件加载为一组文档 
"""
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=300)
docs = text_splitter.split_documents(documents)
"""
实例化 Jaguar 向量存储
"""
### Jaguar HTTP 端点
url = "http://192.168.5.88:8080/fwww/"
### 使用 OpenAI 嵌入模型
embeddings = OpenAIEmbeddings()
### Pod 是向量的数据库
pod = "vdb"
### 向量存储名称
store = "langchain_rag_store"
### 向量索引名称
vector_index = "v"
### 向量索引的类型
# cosine: 距离度量
# fraction: 嵌入向量是小数
# float: 使用浮点数存储的值
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
在 JaguarDB 数据库服务器上创建向量存储。
这只需要运行一次。
"""
# 向量存储的额外元数据字段
metadata = "category char(16)"
# 存储的文本字段的字符数
text_size = 4096
# 在服务器上创建向量存储
vectorstore.create(metadata, text_size)
"""
将文本拆分器中的文本添加到我们的向量存储中
"""
vectorstore.add_documents(docs)
# 或者给文档打标签：
# vectorstore.add_documents(more_docs, text_tag="tags to these documents")
""" 获取检索器对象 """
retriever = vectorstore.as_retriever()
# retriever = vectorstore.as_retriever(search_kwargs={"where": "m1='123' and m2='abc'"})
template = """You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.
Question: {question}
Context: {context}
Answer:
"""
prompt = ChatPromptTemplate.from_template(template)
""" 获取一个大型语言模型 """
LLM = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
""" 创建 RAG 流程的链式结构 """
rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | LLM
    | StrOutputParser()
)
resp = rag_chain.invoke("What did the president say about Justice Breyer?")
print(resp)
```

## 与 Jaguar 向量存储的交互

用户可以直接与 Jaguar 向量存储进行相似性搜索和异常检测。

```python
from langchain_community.vectorstores.jaguar import Jaguar
from langchain_openai import OpenAIEmbeddings
# 实例化 Jaguar 向量存储对象
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
# 登录以进行授权
vectorstore.login()
# 使用两个元数据字段创建向量存储
# 这只需要运行一次。
metadata_str = "author char(32), category char(16)"
vectorstore.create(metadata_str, 1024)
# 添加一组文本
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
# 带过滤条件的搜索（where）
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

抱歉，你似乎忘记输入要翻译的英文段落了。请将需要翻译的英文段落发送给我，我会尽快帮你翻译成中文科普风格的文章。