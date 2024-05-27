# TiDB Vector

[TiDB Cloud](https://tidbcloud.com/) 是一款全面的数据库即服务（DBaaS）解决方案，提供了专用和无服务器选项。TiDB 无服务器现在正在将内置的向量搜索集成到 MySQL 环境中。通过这一增强功能，您可以在 TiDB 无服务器上轻松开发 AI 应用程序，无需新的数据库或额外的技术堆栈。通过加入私人测试版的等待列表，成为第一批体验者，访问网址 https://tidb.cloud/ai。

本笔记本提供了关于如何利用 TiDB Vector 功能的详细指南，展示其特点和实际应用。

## 设置环境

首先安装必要的软件包。

```python
%pip install langchain
%pip install langchain-openai
%pip install pymysql
%pip install tidb-vector
```

配置您将需要的 OpenAI 和 TiDB 主机设置。在本笔记本中，我们将遵循 TiDB Cloud 提供的标准连接方法，以建立安全高效的数据库连接。

```python
# 这里我们使用 import getpass
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
# 从 tidb cloud 控制台复制
tidb_connection_string_template = "mysql+pymysql://<USER>:<PASSWORD>@<HOST>:4000/<DB>?ssl_ca=/etc/ssl/cert.pem&ssl_verify_cert=true&ssl_verify_identity=true"
# tidb_connection_string_template = "mysql+pymysql://root:<PASSWORD>@34.212.137.91:4000/test"
tidb_password = getpass.getpass("Input your TiDB password:")
tidb_connection_string = tidb_connection_string_template.replace(
    "<PASSWORD>", tidb_password
)
```

准备以下数据

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import TiDBVectorStore
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

## 语义相似性搜索

TiDB 支持余弦距离和欧氏距离（'cosine'，'l2'），'cosine' 是默认选择。

下面的代码片段在 TiDB 中创建了一个名为 `TABLE_NAME` 的表，用于优化向量搜索。成功执行此代码后，您将能够直接在 TiDB 数据库中查看和访问 `TABLE_NAME` 表。

```python
TABLE_NAME = "semantic_embeddings"
db = TiDBVectorStore.from_documents(
    documents=docs,
    embedding=embeddings,
    table_name=TABLE_NAME,
    connection_string=tidb_connection_string,
    distance_strategy="cosine",  # 默认值，另一个选项是 "l2"
)
```
```python
query = "What did the president say about Ketanji Brown Jackson"
docs_with_score = db.similarity_search_with_score(query, k=3)
```

请注意，较低的余弦距离表示更高的相似性。

```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```
```output
--------------------------------------------------------------------------------
Score:  0.18459301498220004
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.2172729943284636
A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. 
And if we are to advance liberty and justice, we need to secure the Border and fix the immigration system. 
We can do both. At our border, we’ve installed new technology like cutting-edge scanners to better detect drug smuggling. 
We’ve set up joint patrols with Mexico and Guatemala to catch more human traffickers. 
We’re putting in place dedicated immigration judges so families fleeing persecution and violence can have their cases heard faster. 
We’re securing commitments and supporting partners in South and Central America to host more refugees and secure their own borders.
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.2262166799003692
And for our LGBTQ+ Americans, let’s finally get the bipartisan Equality Act to my desk. The onslaught of state laws targeting transgender Americans and their families is wrong. 
As I said last year, especially to our younger transgender Americans, I will always have your back as your President, so you can be yourself and reach your God-given potential. 
While it often appears that we never agree, that isn’t true. I signed 80 bipartisan bills into law last year. From preventing government shutdowns to protecting Asian-Americans from still-too-common hate crimes to reforming military justice. 
And soon, we’ll strengthen the Violence Against Women Act that I first wrote three decades ago. It is important for us to show the nation that we can come together and do big things. 
So tonight I’m offering a Unity Agenda for the Nation. Four big things we can do together. 
First, beat the opioid epidemic.
```

此外，`similarity_search_with_relevance_scores` 方法可用于获取相关性分数，其中较高的分数表示更大的相似性。

```python
docs_with_relevance_score = db.similarity_search_with_relevance_scores(query, k=2)
for doc, score in docs_with_relevance_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```
```output
--------------------------------------------------------------------------------
Score:  0.8154069850178
今晚，我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人，宪法学者，也是美国最高法院即将退休的大法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一是提名人选担任美国最高法院的法官。
而我在4天前就做到了，当我提名上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律专家之一，将继续布雷耶大法官的卓越传统。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.7827270056715364
曾在私人执业中担任高级诉讼律师。曾担任联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。一个建立共识的人。自提名以来，她得到了广泛的支持——从警察工会到民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民制度。
我们可以做到。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测走私毒品。
我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。
我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭能够更快地得到审理。
我们正在承诺并支持南美和中美的合作伙伴，以接纳更多的难民并保护他们的边界。
--------------------------------------------------------------------------------
```

# 使用元数据进行过滤

使用元数据过滤器执行搜索，以检索与应用的过滤器相符的特定数量的最近邻结果。

## 支持的元数据类型

TiDB Vector Store 中的每个向量都可以与元数据配对，结构化为 JSON 对象中的键值对。键是字符串，值可以是以下类型之一：

- 字符串

- 数字（整数或浮点数）

- 布尔值（true、false）

例如，考虑以下有效的元数据有效载荷：

```json
{
    "page": 12,
    "book_title": "Siddhartha"
}
```

## 元数据过滤器语法

可用的过滤器包括：

- $or - 选择满足任一给定条件的向量。

- $and - 选择满足所有给定条件的向量。

- $eq - 等于

- $ne - 不等于

- $gt - 大于

- $gte - 大于或等于

- $lt - 小于

- $lte - 小于或等于

- $in - 在数组中

- $nin - 不在数组中

假设有一个带有元数据的向量：

```json
{
    "page": 12,
    "book_title": "Siddhartha"
}
```

以下元数据过滤器将匹配该向量：

```json
{"page": 12}
{"page":{"$eq": 12}}
{"page":{"$in": [11, 12, 13]}}
{"page":{"$nin": [13]}}
{"page":{"$lt": 11}}
{
    "$or": [{"page": 11}, {"page": 12}],
    "$and": [{"page": 12}, {"page": 13}],
}
```

请注意，元数据过滤器中的每个键值对都被视为单独的过滤器子句，并且这些子句使用 AND 逻辑运算符组合。

```python
db.add_texts(
    texts=[
        "TiDB Vector 提供先进的高速向量处理能力，通过高效的数据处理和分析支持增强 AI 工作流。",
        "TiDB Vector，基本使用每月低至 10 美元起"
    ],
    metadatas=[
        {"title": "TiDB Vector 功能"},
        {"title": "TiDB Vector 定价"}
    ],
)
```
```output
[UUID('c782cb02-8eec-45be-a31f-fdb78914f0a7'), UUID('08dcd2ba-9f16-4f29-a9b7-18141f8edae3')]
```
```python
docs_with_score = db.similarity_search_with_score(
    "Introduction to TiDB Vector", filter={"title": "TiDB Vector functionality"}, k=4
)
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```
```output
--------------------------------------------------------------------------------
TiDB Vector 提供了先进的高速向量处理能力，通过有效的数据处理和分析支持，增强了 AI 工作流程。
### 作为检索器的使用
在 Langchain 中，检索器是一个接口，用于在对非结构化查询做出响应时检索文档，提供了比向量存储更广泛的功能。下面的代码演示了如何利用 TiDB Vector 作为检索器。
```python
retriever = db.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={"k": 3, "score_threshold": 0.8},
)
docs_retrieved = retriever.invoke(query)
for doc in docs_retrieved:
    print("-" * 80)
    print(doc.page_content)
    print("-" * 80)
```
```output

--------------------------------------------------------------------------------

今晚，我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。还有，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。

今晚，我想向一个致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。

总统最严肃的宪法责任之一是提名某人担任美国最高法院的法官。

4 天前，我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律专家之一，将继续布雷耶司法部长的卓越传统。

--------------------------------------------------------------------------------

```
## 高级使用案例场景
让我们看一个高级使用案例 - 一家旅行社正在为希望拥有特定设施的机场（如干净的休息室和素食选择）的客户定制旅行报告。该过程包括：
- 在机场评论中进行语义搜索，以提取符合这些设施的机场代码。
- 随后的 SQL 查询将这些代码与航线信息进行连接，详细说明与客户偏好相符的航空公司和目的地。
首先，让我们准备一些与机场相关的数据。
```python
# 创建表以存储飞机数据
db.tidb_vector_client.execute(
    """CREATE TABLE airplan_routes (
        id INT AUTO_INCREMENT PRIMARY KEY,
        airport_code VARCHAR(10),
        airline_code VARCHAR(10),
        destination_code VARCHAR(10),
        route_details TEXT,
        duration TIME,
        frequency INT,
        airplane_type VARCHAR(50),
        price DECIMAL(10, 2),
        layover TEXT
    );"""
)
# 向路线和向量表中插入一些数据
db.tidb_vector_client.execute(
    """INSERT INTO airplan_routes (
        airport_code,
        airline_code,
        destination_code,
        route_details,
        duration,
        frequency,
        airplane_type,
        price,
        layover
    ) VALUES 
    ('JFK', 'DL', 'LAX', 'Non-stop from JFK to LAX.', '06:00:00', 5, 'Boeing 777', 299.99, 'None'),
    ('LAX', 'AA', 'ORD', 'Direct LAX to ORD route.', '04:00:00', 3, 'Airbus A320', 149.99, 'None'),
    ('EFGH', 'UA', 'SEA', 'Daily flights from SFO to SEA.', '02:30:00', 7, 'Boeing 737', 129.99, 'None');
    """
)
db.add_texts(
    texts=[
        "Clean lounges and excellent vegetarian dining options. Highly recommended.",
        "Comfortable seating in lounge areas and diverse food selections, including vegetarian.",
        "Small airport with basic facilities.",
    ],
    metadatas=[
        {"airport_code": "JFK"},
        {"airport_code": "LAX"},
        {"airport_code": "EFGH"},
    ],
)
```
```output

[UUID('6dab390f-acd9-4c7d-b252-616606fbc89b'),

 UUID('9e811801-0e6b-4893-8886-60f4fb67ce69'),

 UUID('f426747c-0f7b-4c62-97ed-3eeb7c8dd76e')]

```
通过向量搜索找到具有干净设施和素食选择的机场
```python
retriever = db.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={"k": 3, "score_threshold": 0.85},
)
semantic_query = "你能推荐一个设施干净、提供良好素食餐饮选择的美国机场吗？"
reviews = retriever.invoke(semantic_query)
for r in reviews:
    print("-" * 80)
    print(r.page_content)
    print(r.metadata)
    print("-" * 80)
```
```output

--------------------------------------------------------------------------------

Clean lounges and excellent vegetarian dining options. Highly recommended.

{'airport_code': 'JFK'}

--------------------------------------------------------------------------------

--------------------------------------------------------------------------------

Comfortable seating in lounge areas and diverse food selections, including vegetarian.

{'airport_code': 'LAX'}

--------------------------------------------------------------------------------

```
```python
# 从元数据中提取机场代码
airport_codes = [review.metadata["airport_code"] for review in reviews]
# 执行查询以获取机场详情
search_query = "SELECT * FROM airplan_routes WHERE airport_code IN :codes"
params = {"codes": tuple(airport_codes)}
airport_details = db.tidb_vector_client.execute(search_query, params)
airport_details.get("result")
```
```output

[(1, 'JFK', 'DL', 'LAX', 'JFK到LAX的直达航班。', datetime.timedelta(seconds=21600), 5, '波音777', Decimal('299.99'), '无'),

 (2, 'LAX', 'AA', 'ORD', 'LAX到ORD的直达航线。', datetime.timedelta(seconds=14400), 3, '空中客车A320', Decimal('149.99'), '无')]

```
或者，我们可以通过使用单个 SQL 查询来简化整个过程，以一步完成搜索。
```python
search_query = f"""
    SELECT 
        VEC_Cosine_Distance(se.embedding, :query_vector) as distance, 
        ar.*,
        se.document as airport_review
    FROM 
        airplan_routes ar
    JOIN 
        {TABLE_NAME} se ON ar.airport_code = JSON_UNQUOTE(JSON_EXTRACT(se.meta, '$.airport_code'))
    ORDER BY distance ASC 
    LIMIT 5;
"""
query_vector = embeddings.embed_query(semantic_query)
params = {"query_vector": str(query_vector)}
airport_details = db.tidb_vector_client.execute(search_query, params)
airport_details.get("result")
```
```output

[(0.1219207353407008, 1, 'JFK', 'DL', 'LAX', 'JFK到LAX的直达航班。', datetime.timedelta(seconds=21600), 5, '波音777', Decimal('299.99'), '无', '干净的休息室和出色的素食餐饮选择。强烈推荐。'),

 (0.14613754359804654, 2, 'LAX', 'AA', 'ORD', 'LAX到ORD的直达航线。', datetime.timedelta(seconds=14400), 3, '空中客车A320', Decimal('149.99'), '无', '休息区座椅舒适，食品选择多样，包括素食。'),

 (0.19840519342700513, 3, 'EFGH', 'UA', 'SEA', '从SFO到SEA的每日航班。', datetime.timedelta(seconds=9000), 7, '波音737', Decimal('129.99'), '无', '基本设施的小型机场。')]

```
```python
# 清理
db.tidb_vector_client.execute("DROP TABLE airplan_routes")
```
```output

{'success': True, 'result': 0, 'error': None}

```
# 删除
您可以使用`.drop_vectorstore()`方法来删除 TiDB Vector 存储。
```python
db.drop_vectorstore()
```

