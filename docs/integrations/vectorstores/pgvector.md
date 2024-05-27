# PGVector
> 一个使用 `postgres` 作为后端并利用 `pgvector` 扩展实现 LangChain 向量存储抽象的项目。
该代码位于一个名为 [langchain_postgres](https://github.com/langchain-ai/langchain-postgres/) 的集成包中。
您可以运行以下命令来启动一个带有 `pgvector` 扩展的 postgres 容器：
```shell
docker run --name pgvector-container -e POSTGRES_USER=langchain -e POSTGRES_PASSWORD=langchain -e POSTGRES_DB=langchain -p 6024:5432 -d pgvector/pgvector:pg16
```
## 状态
该代码已从 `langchain_community` 移植到一个名为 `langchain-postgres` 的专用包中。已做出以下更改：
* langchain_postgres 仅适用于 psycopg3。请将连接字符串从 `postgresql+psycopg2://...` 更新为 `postgresql+psycopg://langchain:langchain@...`（是的，驱动程序名称是 `psycopg` 而不是 `psycopg3`，但它将使用 `psycopg3`）。
* 嵌入式存储和集合的模式已更改，以使 `add_documents` 正确工作与用户指定的 id。
* 现在必须显式传递连接对象。
目前，**没有机制**支持在模式更改时进行简单的数据迁移。因此，向量存储中的任何模式更改都需要用户重新创建表并重新添加文档。如果这是一个问题，请使用其他向量存储。如果不是，这个实现对您的用例应该是可以的。
## 安装依赖
在这里，我们使用 `langchain_cohere` 进行嵌入，但您也可以使用其他嵌入提供程序。
```python
!pip install --quiet -U langchain_cohere
!pip install --quiet -U langchain_postgres
```
## 初始化向量存储
```python
from langchain_cohere import CohereEmbeddings
from langchain_core.documents import Document
from langchain_postgres import PGVector
from langchain_postgres.vectorstores import PGVector
# 请参见上面的 docker 命令以启动一个启用了 pgvector 的 postgres 实例。
connection = "postgresql+psycopg://langchain:langchain@localhost:6024/langchain"  # 使用 psycopg3！
collection_name = "my_docs"
embeddings = CohereEmbeddings()
vectorstore = PGVector(
    embeddings=embeddings,
    collection_name=collection_name,
    connection=connection,
    use_jsonb=True,
)
```
## 删除表
如果您需要删除表（例如，将嵌入更新为不同的维度或只是更新嵌入提供程序）：
```python
vectorstore.drop_tables()
```
## 添加文档
向向量存储添加文档：
```python
docs = [
    Document(
        page_content="there are cats in the pond",
        metadata={"id": 1, "location": "pond", "topic": "animals"},
    ),
    Document(
        page_content="ducks are also found in the pond",
        metadata={"id": 2, "location": "pond", "topic": "animals"},
    ),
    Document(
        page_content="fresh apples are available at the market",
        metadata={"id": 3, "location": "market", "topic": "food"},
    ),
    Document(
        page_content="the market also sells fresh oranges",
        metadata={"id": 4, "location": "market", "topic": "food"},
    ),
    Document(
        page_content="the new art exhibit is fascinating",
        metadata={"id": 5, "location": "museum", "topic": "art"},
    ),
    Document(
        page_content="a sculpture exhibit is also at the museum",
        metadata={"id": 6, "location": "museum", "topic": "art"},
    ),
    Document(
        page_content="a new coffee shop opened on Main Street",
        metadata={"id": 7, "location": "Main Street", "topic": "food"},
    ),
    Document(
        page_content="the book club meets at the library",
        metadata={"id": 8, "location": "library", "topic": "reading"},
    ),
    Document(
        page_content="the library hosts a weekly story time for kids",
        metadata={"id": 9, "location": "library", "topic": "reading"},
    ),
    Document(
        page_content="a cooking class for beginners is offered at the community center",
        metadata={"id": 10, "location": "community center", "topic": "classes"},
    ),
]
```
```python
vectorstore.add_documents(docs, ids=[doc.metadata["id"] for doc in docs])
```
```output
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```
```python
vectorstore.similarity_search("kitty", k=10)
```
```output
[Document(page_content='there are cats in the pond', metadata={'id': 1, 'topic': 'animals', 'location': 'pond'}),
 Document(page_content='the book club meets at the library', metadata={'id': 8, 'topic': 'reading', 'location': 'library'}),
 Document(page_content='the library hosts a weekly story time for kids', metadata={'id': 9, 'topic': 'reading', 'location': 'library'}),
 Document(page_content='the new art exhibit is fascinating', metadata={'id': 5, 'topic': 'art', 'location': 'museum'}),
 Document(page_content='ducks are also found in the pond', metadata={'id': 2, 'topic': 'animals', 'location': 'pond'}),
 Document(page_content='the market also sells fresh oranges', metadata={'id': 4, 'topic': 'food', 'location': 'market'}),
 Document(page_content='a cooking class for beginners is offered at the community center', metadata={'id': 10, 'topic': 'classes', 'location': 'community center'}),
 Document(page_content='fresh apples are available at the market', metadata={'id': 3, 'topic': 'food', 'location': 'market'}),
 Document(page_content='a sculpture exhibit is also at the museum', metadata={'id': 6, 'topic': 'art', 'location': 'museum'}),
 Document(page_content='a new coffee shop opened on Main Street', metadata={'id': 7, 'topic': 'food', 'location': 'Main Street'})]
```
通过 ID 添加文档将覆盖任何与该 ID 匹配的现有文档。
```python
docs = [
    Document(
        page_content="池塘里有猫",
        metadata={"id": 1, "location": "pond", "topic": "animals"},
    ),
    Document(
        page_content="池塘里也有鸭子",
        metadata={"id": 2, "location": "pond", "topic": "animals"},
    ),
    Document(
        page_content="市场上有新鲜的苹果",
        metadata={"id": 3, "location": "market", "topic": "food"},
    ),
    Document(
        page_content="市场还出售新鲜的橙子",
        metadata={"id": 4, "location": "market", "topic": "food"},
    ),
    Document(
        page_content="新的艺术展览很迷人",
        metadata={"id": 5, "location": "museum", "topic": "art"},
    ),
    Document(
        page_content="博物馆里也有雕塑展",
        metadata={"id": 6, "location": "museum", "topic": "art"},
    ),
    Document(
        page_content="主街上开了一家新的咖啡店",
        metadata={"id": 7, "location": "Main Street", "topic": "food"},
    ),
    Document(
        page_content="读书俱乐部在图书馆举行会议",
        metadata={"id": 8, "location": "library", "topic": "reading"},
    ),
    Document(
        page_content="图书馆为孩子们举办每周故事时间",
        metadata={"id": 9, "location": "library", "topic": "reading"},
    ),
    Document(
        page_content="社区中心提供初学者烹饪课程",
        metadata={"id": 10, "location": "community center", "topic": "classes"},
    ),
]
```
## 过滤支持
向量存储支持一组可应用于文档元数据的过滤器。
| 运算符 | 含义/类别        |
|--------|-----------------|
| \$eq    | 等于 (==)       |
| \$ne    | 不等于 (!=)     |
| \$lt    | 小于 (<)        |
| \$lte   | 小于或等于 (<=) |
| \$gt    | 大于 (>)        |
| \$gte   | 大于或等于 (>=) |
| \$in    | 特殊情况 (in)   |
| \$nin   | 特殊情况 (not in) |
| \$between | 特殊情况 (between) |
| \$like  | 文本 (like)     |
| \$ilike | 文本 (不区分大小写 like) |
| \$and   | 逻辑 (and)      |
| \$or    | 逻辑 (or)       |
```python
vectorstore.similarity_search("小猫", k=10, filter={"id": {"$in": [1, 5, 2, 9]}})
```
```output
[Document(page_content='池塘里有猫', metadata={'id': 1, 'topic': 'animals', 'location': 'pond'}),
 Document(page_content='图书馆为孩子们举办每周故事时间', metadata={'id': 9, 'topic': 'reading', 'location': 'library'}),
 Document(page_content='新的艺术展览很迷人', metadata={'id': 5, 'topic': 'art', 'location': 'museum'}),
 Document(page_content='池塘里也有鸭子', metadata={'id': 2, 'topic': 'animals', 'location': 'pond'})]
```
如果您提供一个带有多个字段但没有运算符的字典，则顶层将被解释为逻辑 **AND** 过滤器
```python
vectorstore.similarity_search(
    "鸭子",
    k=10,
    filter={"id": {"$in": [1, 5, 2, 9]}, "location": {"$in": ["pond", "market"]}},
)
```
```output
[Document(page_content='池塘里也有鸭子', metadata={'id': 2, 'topic': 'animals', 'location': 'pond'}),
 Document(page_content='池塘里有猫', metadata={'id': 1, 'topic': 'animals', 'location': 'pond'})]
```
```python
vectorstore.similarity_search(
    "鸭子",
    k=10,
    filter={
        "$and": [
            {"id": {"$in": [1, 5, 2, 9]}},
            {"location": {"$in": ["pond", "market"]}},
        ]
    },
)
```
```output
[Document(page_content='池塘里也有鸭子', metadata={'id': 2, 'topic': 'animals', 'location': 'pond'}),
 Document(page_content='池塘里有猫', metadata={'id': 1, 'topic': 'animals', 'location': 'pond'})]
```
```python
vectorstore.similarity_search("鸟", k=10, filter={"location": {"$ne": "pond"}})
```
```output
[Document(page_content='读书俱乐部在图书馆举行会议', metadata={'id': 8, 'topic': 'reading', 'location': 'library'}),
 Document(page_content='新的艺术展览很迷人', metadata={'id': 5, 'topic': 'art', 'location': 'museum'}),
 Document(page_content='图书馆为孩子们举办每周故事时间', metadata={'id': 9, 'topic': 'reading', 'location': 'library'}),
 Document(page_content='博物馆里也有雕塑展', metadata={'id': 6, 'topic': 'art', 'location': 'museum'}),
 Document(page_content='市场还出售新鲜的橙子', metadata={'id': 4, 'topic': 'food', 'location': 'market'}),
```markdown
- 在社区中心提供给初学者的烹饪课程
- 新的咖啡店在主街开业
- 市场有新鲜的苹果可供选购
```