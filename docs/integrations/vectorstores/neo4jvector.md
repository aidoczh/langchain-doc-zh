# Neo4j 向量索引

[Neo4j](https://neo4j.com/) 是一个集成了向量相似度搜索支持的开源图形数据库。

它支持：

- 近似最近邻搜索

- 欧几里得相似度和余弦相似度

- 结合向量和关键词搜索的混合搜索

本笔记展示了如何使用 Neo4j 向量索引 (`Neo4jVector`)。

查看[安装说明](https://neo4j.com/docs/operations-manual/current/installation/)。

```python
# 安装必要的包
%pip install --upgrade --quiet  neo4j
%pip install --upgrade --quiet  langchain-openai
%pip install --upgrade --quiet  tiktoken
```

我们想要使用 `OpenAIEmbeddings`，因此需要获取 OpenAI API 密钥。

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
from langchain_community.vectorstores import Neo4jVector
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

```python
# Neo4jVector 需要 Neo4j 数据库凭据
url = "bolt://localhost:7687"
username = "neo4j"
password = "password"
# 也可以使用环境变量代替直接传递命名参数
# os.environ["NEO4J_URI"] = "bolt://localhost:7687"
# os.environ["NEO4J_USERNAME"] = "neo4j"
# os.environ["NEO4J_PASSWORD"] = "pleaseletmein"
```

## 余弦距离的相似度搜索（默认）

```python
# Neo4jVector 模块将连接到 Neo4j，并在需要时创建向量索引。
db = Neo4jVector.from_documents(
    docs, OpenAIEmbeddings(), url=url, username=username, password=password
)
```

```python
query = "总统对 Ketanji Brown Jackson 说了什么"
docs_with_score = db.similarity_search_with_score(query, k=2)
```

```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("得分: ", score)
    print(doc.page_content)
    print("-" * 80)
```

```output
--------------------------------------------------------------------------------
得分:  0.9076391458511353
今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统的最严肃的宪法责任之一是提名某人担任美国最高法院的法官。
4天前，我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律智慧之一，将继续布雷耶法官的卓越传统。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分:  0.8912242650985718
曾在私人执业中担任高级诉讼律师。曾是联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。一个达成共识的人。自她被提名以来，她得到了广泛的支持——从警察兄弟会到民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民制度。
我们两者都可以做到。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测毒品走私。
我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。
我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭可以更快地审理他们的案件。
我们正在承诺并支持南美和中美的合作伙伴，以接纳更多的难民并保护他们自己的边界。
--------------------------------------------------------------------------------
```

## 使用 vectorstore

上面，我们从头创建了一个 vectorstore。然而，通常我们希望使用现有的 vectorstore。

为了做到这一点，我们可以直接初始化它。

```python
index_name = "vector"  # 默认索引名称
store = Neo4jVector.from_existing_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name=index_name,
)
```

我们还可以使用 `from_existing_graph` 方法从现有图中初始化一个向量存储。该方法从数据库中提取相关的文本信息，计算并将文本嵌入存储回数据库。

```python
# 首先我们在图中创建样本数据
store.query(
    "CREATE (p:Person {name: 'Tomaz', location:'Slovenia', hobby:'Bicycle', age: 33})"
)
```

```output
[]
```

```python
# 现在我们从现有图中初始化
existing_graph = Neo4jVector.from_existing_graph(
    embedding=OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name="person_index",
    node_label="Person",
    text_node_properties=["name", "location"],
    embedding_node_property="embedding",
)
result = existing_graph.similarity_search("Slovenia", k=1)
```

```python
result[0]
```

```output
Document(page_content='\nname: Tomaz\nlocation: Slovenia', metadata={'age': 33, 'hobby': 'Bicycle'})
```

Neo4j 还支持关系向量索引，其中嵌入被存储为关系属性并建立索引。关系向量索引无法通过 LangChain 进行填充，但可以连接到现有的关系向量索引。

```python
# 首先我们在图中创建样本数据和索引
store.query(
    "MERGE (p:Person {name: 'Tomaz'}) "
    "MERGE (p1:Person {name:'Leann'}) "
    "MERGE (p1)-[:FRIEND {text:'example text', embedding:$embedding}]->(p2)",
    params={"embedding": OpenAIEmbeddings().embed_query("example text")},
)
# 创建一个向量索引
relationship_index = "relationship_vector"
store.query(
    """
CREATE VECTOR INDEX $relationship_index
IF NOT EXISTS
FOR ()-[r:FRIEND]-() ON (r.embedding)
OPTIONS {indexConfig: {
 `vector.dimensions`: 1536,
 `vector.similarity_function`: 'cosine'
}}
""",
    params={"relationship_index": relationship_index},
)
```

```output
[]
```

```python
relationship_vector = Neo4jVector.from_existing_relationship_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name=relationship_index,
    text_node_property="text",
)
relationship_vector.similarity_search("Example")
```

```output
[Document(page_content='example text')]
```

### 元数据过滤

Neo4j 向量存储还支持通过结合并行运行时和精确最近邻搜索进行元数据过滤。

_需要 Neo4j 5.18 或更高版本。_

相等过滤具有以下语法。

```python
existing_graph.similarity_search(
    "Slovenia",
    filter={"hobby": "Bicycle", "name": "Tomaz"},
)
```

```output
[Document(page_content='\nname: Tomaz\nlocation: Slovenia', metadata={'age': 33, 'hobby': 'Bicycle'})]
```

元数据过滤还支持以下操作符：

* `$eq: 等于`

* `$ne: 不等于`

* `$lt: 小于`

* `$lte: 小于或等于`

* `$gt: 大于`

* `$gte: 大于或等于`

* `$in: 在值列表中`

* `$nin: 不在值列表中`

* `$between: 介于两个值之间`

* `$like: 文本包含值`

* `$ilike: 小写文本包含值`

```python
existing_graph.similarity_search(
    "Slovenia",
    filter={"hobby": {"$eq": "Bicycle"}, "age": {"$gt": 15}},
)
```

```output
[Document(page_content='\nname: Tomaz\nlocation: Slovenia', metadata={'age': 33, 'hobby': 'Bicycle'})]
```

您还可以在过滤器之间使用 `OR` 操作符。

```python
existing_graph.similarity_search(
    "Slovenia",
    filter={"$or": [{"hobby": {"$eq": "Bicycle"}}, {"age": {"$gt": 15}}]},
)
```

```output
[Document(page_content='\nname: Tomaz\nlocation: Slovenia', metadata={'age': 33, 'hobby': 'Bicycle'})]
```

### 添加文档

我们可以向现有的向量存储中添加文档。

```python
store.add_documents([Document(page_content="foo")])
```

```output
['acbd18db4cc2f85cedef654fccc4a4d8']
```

```python
docs_with_score = store.similarity_search_with_score("foo")
```

```python
docs_with_score[0]
```

```output
(Document(page_content='foo'), 0.9999997615814209)
```

## 使用检索查询自定义响应

您还可以通过使用自定义的 Cypher 片段来自定义响应，该片段可以从图中获取其他信息。

在幕后，最终的 Cypher 语句构造如下：

```
read_query = (
  "CALL db.index.vector.queryNodes($index, $k, $embedding) "
  "YIELD node, score "
) + retrieval_query
```

检索查询必须返回以下三列：

* `text`: Union[str, Dict] = 用于填充文档的 `page_content` 的值

* `score`: Float = 相似度分数

* `metadata`: Dict = 文档的附加元数据

在这篇[博客文章](https://medium.com/neo4j/implementing-rag-how-to-write-a-graph-retrieval-query-in-langchain-74abf13044f2)中了解更多。

```python
retrieval_query = """
RETURN "Name:" + node.name AS text, score, {foo:"bar"} AS metadata
"""
retrieval_example = Neo4jVector.from_existing_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name="person_index",
    retrieval_query=retrieval_query,
)
retrieval_example.similarity_search("Foo", k=1)
```

```output
[Document(page_content='Name:Tomaz', metadata={'foo': 'bar'})]
```

这是一个将除了 `embedding` 之外的所有节点属性作为字典传递给 `text` 列的示例，

```python
retrieval_query = """
RETURN node {.name, .age, .hobby} AS text, score, {foo:"bar"} AS metadata
"""
retrieval_example = Neo4jVector.from_existing_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name="person_index",
    retrieval_query=retrieval_query,
)
retrieval_example.similarity_search("Foo", k=1)
```

```output
[Document(page_content='name: Tomaz\nage: 33\nhobby: Bicycle\n', metadata={'foo': 'bar'})]
```

您还可以将 Cypher 参数传递给检索查询。

参数可用于额外的过滤、遍历等...

```python
retrieval_query = """
RETURN node {.*, embedding:Null, extra: $extra} AS text, score, {foo:"bar"} AS metadata
"""
retrieval_example = Neo4jVector.from_existing_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name="person_index",
    retrieval_query=retrieval_query,
)
retrieval_example.similarity_search("Foo", k=1, params={"extra": "ParamInfo"})
```

```output
[Document(page_content='location: Slovenia\nextra: ParamInfo\nname: Tomaz\nage: 33\nhobby: Bicycle\nembedding: None\n', metadata={'foo': 'bar'})]
```

## 混合搜索（向量 + 关键词）

Neo4j 集成了向量和关键词索引，这使您可以使用混合搜索方法

```python
# Neo4jVector 模块将连接到 Neo4j 并在需要时创建向量和关键词索引。
hybrid_db = Neo4jVector.from_documents(
    docs,
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    search_type="hybrid",
)
```

要从现有索引加载混合搜索，您必须提供向量和关键词索引

```python
index_name = "vector"  # 默认索引名称
keyword_index_name = "keyword"  # 默认关键词索引名称
store = Neo4jVector.from_existing_index(
    OpenAIEmbeddings(),
    url=url,
    username=username,
    password=password,
    index_name=index_name,
    keyword_index_name=keyword_index_name,
    search_type="hybrid",
)
```

## 检索器选项

本节介绍如何将 `Neo4jVector` 用作检索器。

```python
retriever = store.as_retriever()
retriever.invoke(query)[0]
```

```output
Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../how_to/state_of_the_union.txt'})
```

## 使用来源进行问答

本节介绍如何使用索引进行带来源的问答。它通过使用 `RetrievalQAWithSourcesChain` 实现，该链将从索引中查找文档。

```python
from langchain.chains import RetrievalQAWithSourcesChain
from langchain_openai import ChatOpenAI
```

```python
chain = RetrievalQAWithSourcesChain.from_chain_type(
    ChatOpenAI(temperature=0), chain_type="stuff", retriever=retriever
)
```

```python
chain.invoke(
    {"question": "What did the president say about Justice Breyer"},
    return_only_outputs=True,
)
```

```output
{'answer': 'The president honored Justice Stephen Breyer for his service to the country and mentioned his retirement from the United States Supreme Court.\n',
 'sources': '../../how_to/state_of_the_union.txt'}
```

