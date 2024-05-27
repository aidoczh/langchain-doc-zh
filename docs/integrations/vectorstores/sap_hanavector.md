# SAP HANA 云矢量引擎
[SAP HANA 云矢量引擎](https://www.sap.com/events/teched/news-guide/ai.html#article8) 是完全集成到 `SAP HANA 云` 数据库中的矢量存储。
## 设置
安装 HANA 数据库驱动程序。
```python
# 安装必要的包
%pip install --upgrade --quiet  hdbcli
```
对于 `OpenAIEmbeddings`，我们使用环境中的 OpenAI API 密钥。
```python
import os
# 使用 OPENAI_API_KEY 环境变量
# os.environ["OPENAI_API_KEY"] = "Your OpenAI API key"
```
创建到 HANA 云实例的数据库连接。
```python
from hdbcli import dbapi
# 使用环境中的连接设置
connection = dbapi.connect(
    address=os.environ.get("HANA_DB_ADDRESS"),
    port=os.environ.get("HANA_DB_PORT"),
    user=os.environ.get("HANA_DB_USER"),
    password=os.environ.get("HANA_DB_PASSWORD"),
    autocommit=True,
    sslValidateCertificate=False,
)
```
## 示例
加载示例文档 "state_of_the_union.txt" 并从中创建块。
```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.hanavector import HanaDB
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
text_documents = TextLoader("../../how_to/state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=0)
text_chunks = text_splitter.split_documents(text_documents)
print(f"Number of document chunks: {len(text_chunks)}")
embeddings = OpenAIEmbeddings()
```
为 HANA 数据库创建 LangChain VectorStore 接口，并指定用于访问矢量嵌入的表（集合）。
```python
db = HanaDB(
    embedding=embeddings, connection=connection, table_name="STATE_OF_THE_UNION"
)
```
将加载的文档块添加到表中。在此示例中，我们删除表中可能存在的先前运行的任何内容。
```python
# 从表中删除已存在的文档
db.delete(filter={})
# 添加加载的文档块
db.add_documents(text_chunks)
```
执行查询以从前一步骤中添加的文档块中获取两个最佳匹配的文档块。默认情况下，使用 "余弦相似度" 进行搜索。
```python
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = db.similarity_search(query, k=2)
for doc in docs:
    print("-" * 80)
    print(doc.page_content)
```
使用 "欧几里得距离" 查询相同内容。结果应该与 "余弦相似度" 相同。
```python
from langchain_community.vectorstores.utils import DistanceStrategy
db = HanaDB(
    embedding=embeddings,
    connection=connection,
    distance_strategy=DistanceStrategy.EUCLIDEAN_DISTANCE,
    table_name="STATE_OF_THE_UNION",
)
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = db.similarity_search(query, k=2)
for doc in docs:
    print("-" * 80)
    print(doc.page_content)
```
## 最大边际相关搜索（MMR）
`最大边际相关` 优化了与查询的相似性和所选文档之间的多样性。将从数据库中检索前 20 个（fetch_k）项目。然后 MMR 算法将找到最佳的 2（k）个匹配项。
```python
docs = db.max_marginal_relevance_search(query, k=2, fetch_k=20)
for doc in docs:
    print("-" * 80)
    print(doc.page_content)
```
## 基本 Vectorstore 操作
```python
db = HanaDB(
    connection=connection, embedding=embeddings, table_name="LANGCHAIN_DEMO_BASIC"
)
# 从表中删除已存在的文档
db.delete(filter={})
```
我们可以将简单的文本文档添加到现有表中。
```python
docs = [Document(page_content="一些文本"), Document(page_content="其他文档")]
db.add_documents(docs)
```
添加带有元数据的文档。
```python
docs = [
    Document(
        page_content="foo",
        metadata={"start": 100, "end": 150, "doc_name": "foo.txt", "quality": "bad"},
    ),
    Document(
        page_content="bar",
        metadata={"start": 200, "end": 250, "doc_name": "bar.txt", "quality": "good"},
    ),
]
db.add_documents(docs)
```
使用特定元数据查询文档。
```python
docs = db.similarity_search("foobar", k=2, filter={"quality": "bad"})
# 使用 "quality"=="bad" 进行过滤，应该只返回一个文档
for doc in docs:
    print("-" * 80)
    print(doc.page_content)
    print(doc.metadata)
```
删除具有特定元数据的文档。
```python
db.delete(filter={"quality": "bad"})
# 现在使用相同的过滤器进行相似性搜索将不返回结果
docs = db.similarity_search("foobar", k=2, filter={"quality": "bad"})
print(len(docs))
```
## 高级过滤
除了基本的基于数值的过滤功能外，还可以使用更高级的过滤功能。下表显示了可用的过滤操作符。
| 操作符 | 语义                 |
|----------|-------------------------|
| `$eq`    | 等于 (==)           |
| `$ne`    | 不等于 (!=)         |
| `$lt`    | 小于 (<)           |
| `$lte`   | 小于或等于 (<=) |
| `$gt`    | 大于 (>)        |
| `$gte`   | 大于或等于 (>=) |
| `$in`    | 包含在给定值集合中（in）    |
| `$nin`   | 不包含在给定值集合中（not in）  |
| `$between` | 两个边界值之间的范围 |
| `$like`  | 基于 SQL 中 "LIKE" 语义的文本相等性（使用 "%" 作为通配符）  |
| `$and`   | 逻辑 "与"，支持 2 个或更多操作数 |
| `$or`    | 逻辑 "或"，支持 2 个或更多操作数 |
```python
# 准备一些测试文档
docs = [
    Document(
        page_content="First",
        metadata={"name": "adam", "is_active": True, "id": 1, "height": 10.0},
    ),
    Document(
        page_content="Second",
        metadata={"name": "bob", "is_active": False, "id": 2, "height": 5.7},
    ),
    Document(
        page_content="Third",
        metadata={"name": "jane", "is_active": True, "id": 3, "height": 2.4},
    ),
]
db = HanaDB(
    connection=connection,
    embedding=embeddings,
    table_name="LANGCHAIN_DEMO_ADVANCED_FILTER",
)
# 从表中删除已存在的文档
db.delete(filter={})
db.add_documents(docs)
# 用于打印过滤结果的辅助函数
def print_filter_result(result):
    if len(result) == 0:
        print("<empty result>")
    for doc in result:
        print(doc.metadata)
```
使用 `$ne`, `$gt`, `$gte`, `$lt`, `$lte` 进行过滤
```python
advanced_filter = {"id": {"$ne": 1}}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
advanced_filter = {"id": {"$gt": 1}}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
advanced_filter = {"id": {"$gte": 1}}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
advanced_filter = {"id": {"$lt": 1}}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
advanced_filter = {"id": {"$lte": 1}}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
```
使用 `$between`, `$in`, `$nin` 进行过滤
```python
advanced_filter = {"id": {"$between": (1, 2)}}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
advanced_filter = {"name": {"$in": ["adam", "bob"]}}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
advanced_filter = {"name": {"$nin": ["adam", "bob"]}}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
```
使用 `$like` 进行文本过滤
```python
advanced_filter = {"name": {"$like": "a%"}}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
advanced_filter = {"name": {"$like": "%a%"}}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
```
使用 `$and`, `$or` 进行组合过滤
```python
advanced_filter = {"$or": [{"id": 1}, {"name": "bob"}]}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
advanced_filter = {"$and": [{"id": 1}, {"id": 2}]}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
advanced_filter = {"$or": [{"id": 1}, {"id": 2}, {"id": 3}]}
print(f"过滤器: {advanced_filter}")
print_filter_result(db.similarity_search("just testing", k=5, filter=advanced_filter))
```
## 在检索增强生成（RAG）中使用 VectorStore 作为检索器链的检索器
```python
from langchain.memory import ConversationBufferMemory
from langchain_openai import ChatOpenAI
# 使用新表访问向量数据库
db = HanaDB(
    connection=connection,
    embedding=embeddings,
    table_name="LANGCHAIN_DEMO_RETRIEVAL_CHAIN",
)
# 从表中删除已存在的条目
db.delete(filter={})
# 添加从“国情咨文”文件加载的文档块
db.add_documents(text_chunks)
# 创建向量存储的检索器实例
retriever = db.as_retriever()
```
定义提示
```python
from langchain_core.prompts import PromptTemplate
prompt_template = """
您是国情咨文专家。您提供了与您必须回答的提示相关的多个上下文项。
使用以下上下文片段回答最后的问题。
```
```markdown
{context}
```
问题：{question}
```
PROMPT = PromptTemplate(
    template=prompt_template, input_variables=["context", "question"]
)
chain_type_kwargs = {"prompt": PROMPT}
```
创建 ConversationalRetrievalChain，处理聊天历史记录和检索类似文档片段以添加到提示中。
```python
from langchain.chains import ConversationalRetrievalChain
llm = ChatOpenAI(model="gpt-3.5-turbo")
memory = ConversationBufferMemory(
    memory_key="chat_history", output_key="answer", return_messages=True
)
qa_chain = ConversationalRetrievalChain.from_llm(
    llm,
    db.as_retriever(search_kwargs={"k": 5}),
    return_source_documents=True,
    memory=memory,
    verbose=False,
    combine_docs_chain_kwargs={"prompt": PROMPT},
)
```
提出第一个问题（并验证已使用多少文本片段）。
```python
question = "关于墨西哥和危地马拉呢？"
result = qa_chain.invoke({"question": question})
print("LLM 的回答：")
print("================")
print(result["answer"])
source_docs = result["source_documents"]
print("================")
print(f"已使用的源文档片段数量：{len(source_docs)}")
```
详细检查链中使用的片段。检查排名最高的片段是否包含与问题中提到的“墨西哥和危地马拉”相关的信息。
```python
for doc in source_docs:
    print("-" * 80)
    print(doc.page_content)
    print(doc.metadata)
```
在同一对话链上提出另一个问题。答案应与先前给出的答案相关。
```python
question = "其他国家呢？"
result = qa_chain.invoke({"question": question})
print("LLM 的回答：")
print("================")
print(result["answer"])
```
## 标准表格 vs. 带有向量数据的“自定义”表格
作为默认行为，嵌入的表格创建了3列：
- 一个名为 `VEC_TEXT` 的列，其中包含文档的文本
- 一个名为 `VEC_META` 的列，其中包含文档的元数据
- 一个名为 `VEC_VECTOR` 的列，其中包含文档文本的嵌入向量
```python
# 使用新表格访问向量数据库
db = HanaDB(
    connection=connection, embedding=embeddings, table_name="LANGCHAIN_DEMO_NEW_TABLE"
)
# 从表格中删除已存在的条目
db.delete(filter={})
# 添加一份包含一些元数据的简单文档
docs = [
    Document(
        page_content="一份简单的文档",
        metadata={"start": 100, "end": 150, "doc_name": "simple.txt"},
    )
]
db.add_documents(docs)
```
显示表格 "LANGCHAIN_DEMO_NEW_TABLE" 中的列
```python
cur = connection.cursor()
cur.execute(
    "SELECT COLUMN_NAME, DATA_TYPE_NAME FROM SYS.TABLE_COLUMNS WHERE SCHEMA_NAME = CURRENT_SCHEMA AND TABLE_NAME = 'LANGCHAIN_DEMO_NEW_TABLE'"
)
rows = cur.fetchall()
for row in rows:
    print(row)
cur.close()
```
显示插入文档在三列中的值
```python
cur = connection.cursor()
cur.execute(
    "SELECT VEC_TEXT, VEC_META, TO_NVARCHAR(VEC_VECTOR) FROM LANGCHAIN_DEMO_NEW_TABLE LIMIT 1"
)
rows = cur.fetchall()
print(rows[0][0])  # 文本
print(rows[0][1])  # 元数据
print(rows[0][2])  # 向量
cur.close()
```
自定义表格必须至少有三列，与标准表格的语义相匹配
- 一列类型为 `NCLOB` 或 `NVARCHAR`，用于嵌入的文本/上下文
- 一列类型为 `NCLOB` 或 `NVARCHAR`，用于元数据 
- 一列类型为 `REAL_VECTOR`，用于嵌入向量
表格可以包含额外的列。当新文档插入表格时，这些额外的列必须允许 NULL 值。
```python
# 创建一个名为 "MY_OWN_TABLE" 的新表格，包含三个“标准”列和一个额外的列
my_own_table_name = "MY_OWN_TABLE"
cur = connection.cursor()
cur.execute(
    (
        f"CREATE TABLE {my_own_table_name} ("
        "SOME_OTHER_COLUMN NVARCHAR(42), "
        "MY_TEXT NVARCHAR(2048), "
        "MY_METADATA NVARCHAR(1024), "
        "MY_VECTOR REAL_VECTOR )"
    )
)
# 使用自定义表格创建一个 HanaDB 实例
db = HanaDB(
    connection=connection,
    embedding=embeddings,
    table_name=my_own_table_name,
    content_column="MY_TEXT",
    metadata_column="MY_METADATA",
    vector_column="MY_VECTOR",
)
# 添加一份包含一些元数据的简单文档
docs = [
    Document(
        page_content="一些其他文本",
        metadata={"start": 400, "end": 450, "doc_name": "other.txt"},
    )
]
db.add_documents(docs)
# 检查数据是否已插入到我们自己的表格中
cur.execute(f"SELECT * FROM {my_own_table_name} LIMIT 1")
rows = cur.fetchall()
print(rows[0][0])  # 列 "SOME_OTHER_DATA" 的值。应为 NULL/None
print(rows[0][1])  # 文本
print(rows[0][2])  # 元数据
print(rows[0][3])  # 向量
cur.close()
```
添加另一份文档并在自定义表格上执行相似性搜索。
```python
docs = [
    Document(
        page_content="一些更多的文本",
        metadata={"start": 800, "end": 950, "doc_name": "more.txt"},
    )
]
db.add_documents(docs)
query = "最近怎么样？"
docs = db.similarity_search(query, k=2)
for doc in docs:
    print("-" * 80)
    print(doc.page_content)
```