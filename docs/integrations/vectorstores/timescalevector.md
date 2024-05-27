# Timescale Vector (Postgres)

>[Timescale Vector](https://www.timescale.com/ai?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 是用于 AI 应用的 `PostgreSQL++` 向量数据库。

这篇笔记展示了如何使用 Postgres 向量数据库 `Timescale Vector`。您将学习如何使用 Timescale Vector 进行 (1) 语义搜索，(2) 基于时间的向量搜索，(3) 自查询，以及 (4) 如何创建索引以加快查询速度。

## 什么是 Timescale Vector?

`Timescale Vector` 能够在 `PostgreSQL` 中高效地存储和查询数百万个向量嵌入。

- 通过受 `DiskANN` 启发的索引算法，提升了 `pgvector` 在 1 亿个以上向量上的相似性搜索速度和准确性。

- 通过自动的基于时间的分区和索引，实现了快速的基于时间的向量搜索。

- 提供了熟悉的 SQL 接口，用于查询向量嵌入和关系数据。

`Timescale Vector` 是一种可随着您的需求从 POC 到生产环境进行扩展的云端 AI `PostgreSQL`：

- 通过使您能够在单个数据库中存储关系元数据、向量嵌入和时间序列数据，简化了操作。

- 借助企业级功能，如流式备份和复制、高可用性和行级安全性，从坚实的 PostgreSQL 基础中获益。

- 通过企业级安全性和合规性，实现了无忧体验。

## 如何访问 Timescale Vector

`Timescale Vector` 可在 [Timescale](https://www.timescale.com/ai?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 平台上使用，这是一个云端 PostgreSQL 平台。（目前没有自托管版本。）

LangChain 用户可获得 Timescale Vector 的 90 天免费试用。

- 要开始使用，[注册](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) Timescale，创建一个新数据库，然后按照本笔记！

- 查看 [Timescale Vector 解释博客](https://www.timescale.com/blog/how-we-made-postgresql-the-best-vector-database/?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 以获取更多详细信息和性能基准。

- 查看 [安装说明](https://github.com/timescale/python-vector) 以获取有关在 Python 中使用 Timescale Vector 的更多详细信息。

## 设置

按照以下步骤准备好跟随本教程。

```python
# 安装必要的包
%pip install --upgrade --quiet  timescale-vector
%pip install --upgrade --quiet  langchain-openai
%pip install --upgrade --quiet  tiktoken
```

在本示例中，我们将使用 `OpenAIEmbeddings`，因此让我们加载您的 OpenAI API 密钥。

```python
import os
from dotenv import find_dotenv, load_dotenv
_ = load_dotenv(find_dotenv())
OPENAI_API_KEY = os.environ["OPENAI_API_KEY"]
```

```python
from typing import Tuple
```

接下来，我们将导入所需的 Python 库和 LangChain 库。请注意，我们导入了 `timescale-vector` 库以及 TimescaleVector LangChain 向量存储库。

```python
from datetime import datetime, timedelta
from langchain_community.document_loaders import TextLoader
from langchain_community.document_loaders.json_loader import JSONLoader
from langchain_community.vectorstores.timescalevector import TimescaleVector
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

## 1. 使用欧氏距离进行相似性搜索（默认）

首先，我们将看一个示例，对国情咨文进行相似性搜索查询，以找到与给定查询句子最相似的句子。我们将使用 [欧氏距离](https://en.wikipedia.org/wiki/Euclidean_distance) 作为我们的相似性度量。

```python
# 加载文本并将其分成块
loader = TextLoader("../../../extras/modules/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

接下来，我们将加载 Timescale 数据库的服务 URL。

如果尚未这样做，请[注册 Timescale](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral)，并创建一个新数据库。

然后，要连接到您的 PostgreSQL 数据库，您需要您的服务 URI，该 URI 可以在创建新数据库后下载的 cheatsheet 或 `.env` 文件中找到。

URI 将类似于这样：`postgres://tsdbadmin:<password>@<id>.tsdb.cloud.timescale.com:<port>/tsdb?sslmode=require`。

```python
# Timescale Vector 需要您的云数据库的服务 URL。您可以在创建服务后立即在云 UI 中或下载的 credentials.sql 文件中找到此 URL
SERVICE_URL = os.environ["TIMESCALE_SERVICE_URL"]
# 如果进行测试，可以直接指定
# SERVICE_URL = "postgres://tsdbadmin:<password>@<id>.tsdb.cloud.timescale.com:<port>/tsdb?sslmode=require"
# 也可以从环境变量中获取。我们建议使用 .env 文件。
# import os
# SERVICE_URL = os.environ.get("TIMESCALE_SERVICE_URL", "")
```

接下来我们创建一个 TimescaleVector vectorstore。我们指定一个集合名称，这将是我们存储数据的表的名称。

注意：当创建一个新的 TimescaleVector 实例时，TimescaleVector 模块将尝试创建一个以集合名称命名的表。因此，请确保集合名称是唯一的（即它不存在）。

```python
# TimescaleVector 模块将创建一个以集合名称命名的表。
COLLECTION_NAME = "state_of_the_union_test"
# 从文档集合创建一个 Timescale Vector 实例
db = TimescaleVector.from_documents(
    embedding=embeddings,
    documents=docs,
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
)
```

现在我们已经加载了我们的数据，我们可以执行相似性搜索。

```python
query = "总统关于 Ketanji Brown Jackson 说了什么"
docs_with_score = db.similarity_search_with_score(query)
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
得分:  0.18443380687035138
今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。而且在此期间，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一就是提名某人担任美国最高法院大法官。
而我在4天前就做到了，当时我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续延续布雷耶司法部长的卓越传统。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分:  0.18452197313308139
今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。而且在此期间，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一就是提名某人担任美国最高法院大法官。
而我在4天前就做到了，当时我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续延续布雷耶司法部长的卓越传统。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分:  0.21720781018594182
曾是私人执业的顶尖诉讼律师。曾是联邦公共辩护律师。来自一家公立学校教育工作者和警察的家庭。一个达成共识的人。自提名以来，她得到了广泛的支持——从警察兄弟会到民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民制度。
我们两者都可以做到。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测毒品走私。
我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。
我们正在设立专门的移民法官，以便那些逃离迫害和暴力的家庭能够更快地得到审理。
我们正在确保承诺并支持南美和中美的合作伙伴，以接纳更多的难民并保护他们自己的边界。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分:  0.21724902288621384
曾是私人执业的顶尖诉讼律师。曾是联邦公共辩护律师。来自一家公立学校教育工作者和警察的家庭。一个达成共识的人。自提名以来，她得到了广泛的支持——从警察兄弟会到民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民制度。
我们两者都可以做到。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测毒品走私。
我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。
```

我们正在设立专门的移民法官，以便那些逃离迫害和暴力的家庭能够更快地得到审理。

我们正在确保南美和中美洲的合作伙伴承诺并支持接纳更多的难民，并保护他们自己的边界。

---

### 使用时间向量作为检索器

在初始化时间向量存储之后，您可以将其用作[检索器](/docs/how_to#retrievers)。

```python
# 使用时间向量作为检索器
retriever = db.as_retriever()
```

```python
print(retriever)
```

```output
tags=['TimescaleVector', 'OpenAIEmbeddings'] metadata=None vectorstore=<langchain_community.vectorstores.timescalevector.TimescaleVector object at 0x10fc8d070> search_type='similarity' search_kwargs={}
```

让我们看一个使用时间向量作为检索器的示例，结合 RetrievalQA 链和 stuff 文档链。

在这个例子中，我们将询问与上面相同的查询，但这次我们将从时间向量返回的相关文档传递给 LLM 作为上下文来回答我们的问题。

首先，我们将创建我们的 stuff 链：

```python
# 初始化 GPT3.5 模型
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(temperature=0.1, model="gpt-3.5-turbo-16k")
# 从 stuff 链创建一个 RetrievalQA 类
from langchain.chains import RetrievalQA
qa_stuff = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=retriever,
    verbose=True,
)
```

```python
query = "总统对 Ketanji Brown Jackson 有什么说？"
response = qa_stuff.run(query)
```

```output
> 进入新的 RetrievalQA 链...
> 链完成。
```

```python
print(response)
```

```output
总统表示，他提名了巡回上诉法院法官 Ketanji Brown Jackson，她是我们国家顶尖的法律专家之一，将继续布雷尔大法官的卓越传统。他还提到自提名以来，她得到了来自各种团体的广泛支持，包括警察兄弟会和民主党和共和党任命的前法官。
```

## 2. 基于时间的相似性搜索

时间向量的一个关键用例是高效的基于时间的向量搜索。时间向量通过自动按时间对向量（和相关元数据）进行分区来实现这一点。这使您能够通过相似性和时间高效地查询向量。

基于时间的向量搜索功能对于以下应用非常有帮助：

- 存储和检索 LLM 响应历史（例如聊天机器人）

- 查找与查询向量相似的最新嵌入（例如最新新闻）

- 将相似性搜索限制在相关的时间范围内（例如关于知识库的基于时间的问题）

为了说明如何使用时间向量的基于时间的向量搜索功能，我们将询问关于 TimescaleDB 的 git 日志历史的问题。我们将演示如何添加带有基于时间的 uuid 的文档以及如何在时间范围内运行相似性搜索。

### 从 git log JSON 中提取内容和元数据

首先，让我们将 git log 数据加载到我们的 PostgreSQL 数据库中的一个名为 `timescale_commits` 的新集合中。

我们将定义一个辅助函数来为文档和相关的向量嵌入创建一个基于时间戳的 uuid。我们将使用这个函数为每个 git log 条目创建一个 uuid。

重要提示：如果您正在处理文档并希望将当前日期和时间与用于基于时间的搜索的向量关联起来，您可以跳过此步骤。默认情况下，文档被摄取时将自动生成一个 uuid。

```python
from timescale_vector import client
# 用于接受过去日期字符串并返回 uuid v1 的函数
def create_uuid(date_string: str):
    if date_string is None:
        return None
    time_format = "%a %b %d %H:%M:%S %Y %z"
    datetime_obj = datetime.strptime(date_string, time_format)
    uuid = client.uuid_from_time(datetime_obj)
    return str(uuid)
```

接下来，我们将定义一个元数据函数，从 JSON 记录中提取相关的元数据。我们将把这个函数传递给 JSONLoader。有关更多详细信息，请参阅[JSON 文档加载器文档](/docs/how_to/document_loader_json)。

```python
# 定义一个字典，将月份的缩写映射到它们的数字等价物上
month_dict = {
    "Jan": "01",
    "Feb": "02",
    "Mar": "03",
    "Apr": "04",
    "May": "05",
    "Jun": "06",
    "Jul": "07",
    "Aug": "08",
    "Sep": "09",
    "Oct": "10",
    "Nov": "11",
    "Dec": "12",
}
# 将输入字符串分割成各个组成部分
components = input_string.split()
# 提取相关信息
day = components[2]
month = month_dict[components[1]]
year = components[4]
time = components[3]
timezone_offset_minutes = int(components[5])  # 将偏移量转换为分钟
timezone_hours = timezone_offset_minutes // 60  # 计算小时
timezone_minutes = timezone_offset_minutes % 60  # 计算剩余的分钟
# 创建一个符合 PostgreSQL 格式的 timestamptz 格式化字符串
timestamp_tz_str = (
    f"{year}-{month}-{day} {time}+{timezone_hours:02}{timezone_minutes:02}"
)
return timestamp_tz_str
# 从 JSON 记录中提取元数据的元数据提取函数
def extract_metadata(record: dict, metadata: dict) -> dict:
    record_name, record_email = split_name(record["author"])
    metadata["id"] = create_uuid(record["date"])
    metadata["date"] = create_date(record["date"])
    metadata["author_name"] = record_name
    metadata["author_email"] = record_email
    metadata["commit_hash"] = record["commit"]
    return metadata
```

接下来，您需要[下载示例数据集](https://s3.amazonaws.com/assets.timescale.com/ai/ts_git_log.json) 并将其放置在与此笔记本相同的目录中。

您可以使用以下命令：

```python
# 使用 curl 下载文件并将其保存为 commit_history.csv
# 注意：在终端中执行此命令，要在与笔记本相同的目录中执行
!curl -O https://s3.amazonaws.com/assets.timescale.com/ai/ts_git_log.json
```

最后，我们可以初始化 JSON 加载器以解析 JSON 记录。我们还会删除空记录以简化操作。

```python
# 定义相对于此笔记本的 JSON 文件路径
# 将此路径更改为您的 JSON 文件路径
FILE_PATH = "../../../../../ts_git_log.json"
# 从 JSON 文件加载数据并提取元数据
loader = JSONLoader(
    file_path=FILE_PATH,
    jq_schema=".commit_history[]",
    text_content=False,
    metadata_func=extract_metadata,
)
documents = loader.load()
# 删除日期为 None 的文档
documents = [doc for doc in documents if doc.metadata["date"] is not None]
```

```python
print(documents[0])
```

```output
page_content='{"commit": "44e41c12ab25e36c202f58e068ced262eadc8d16", "author": "Lakshmi Narayanan Sreethar<lakshmi@timescale.com>", "date": "Tue Sep 5 21:03:21 2023 +0530", "change summary": "Fix segfault in set_integer_now_func", "change details": "When an invalid function oid is passed to set_integer_now_func, it finds out that the function oid is invalid but before throwing the error, it calls ReleaseSysCache on an invalid tuple causing a segfault. Fixed that by removing the invalid call to ReleaseSysCache.  Fixes #6037 "}' metadata={'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/ts_git_log.json', 'seq_num': 1, 'id': '8b407680-4c01-11ee-96a6-b82284ddccc6', 'date': '2023-09-5 21:03:21+0850', 'author_name': 'Lakshmi Narayanan Sreethar', 'author_email': 'lakshmi@timescale.com', 'commit_hash': '44e41c12ab25e36c202f58e068ced262eadc8d16'}
```

### 将文档和元数据加载到 TimescaleVector 向量存储中

现在我们已经准备好我们的文档，让我们处理它们并将它们以及它们的向量嵌入表示加载到我们的 TimescaleVector 向量存储中。

由于这是一个演示，我们只会加载前 500 条记录。在实际应用中，您可以加载任意数量的记录。

```python
NUM_RECORDS = 500
documents = documents[:NUM_RECORDS]
```

然后，我们使用 CharacterTextSplitter 将文档拆分成较小的块，以便更容易进行嵌入。请注意，此拆分过程保留了每个文档的元数据。

```python
# 将文档拆分成块以进行嵌入
text_splitter = CharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
)
docs = text_splitter.split_documents(documents)
```

接下来，我们将从我们完成的预处理文档集合中创建一个 Timescale Vector 实例。

首先，我们将定义一个集合名称，这将是我们在 PostgreSQL 数据库中的表的名称。

我们还将定义一个时间间隔，将其传递给 `time_partition_interval` 参数，该参数将用于按时间对数据进行分区。每个分区将包含指定长度时间的数据。为了简单起见，我们将使用 7 天，但您可以根据您的用例选择任何合适的值 -- 例如，如果您经常查询最近的向量，则可能希望使用较小的时间间隔，如 1 天；或者如果您要查询长达十年的向量，则可能希望使用较大的时间间隔，如 6 个月或 1 年。

最后，我们将创建 TimescaleVector 实例。我们将指定 `ids` 参数为我们在上面的预处理步骤中创建的元数据中的 `uuid` 字段。我们这样做是因为我们希望我们的 uuid 的时间部分反映过去的日期（即提交时的日期）。但是，如果我们希望当前日期和时间与我们的文档关联，我们可以删除 id 参数，uuid 将自动使用当前日期和时间创建。

```python
# 定义集合名称
COLLECTION_NAME = "timescale_commits"
embeddings = OpenAIEmbeddings()
# 从文档集合中创建一个 Timescale Vector 实例
db = TimescaleVector.from_documents(
    embedding=embeddings,
    ids=[doc.metadata["id"] for doc in docs],
    documents=docs,
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
    time_partition_interval=timedelta(days=7),
)
```

### 按时间和相似度查询向量

现在我们已经将文档加载到 TimescaleVector 中，我们可以通过时间和相似度对它们进行查询。

TimescaleVector 提供了多种方法来通过时间进行相似度搜索。

让我们来看看下面的每种方法：

```python
# 时间过滤变量
start_dt = datetime(2023, 8, 1, 22, 10, 35)  # 开始日期 = 2023年8月1日，22:10:35
end_dt = datetime(2023, 8, 30, 22, 10, 35)  # 结束日期 = 2023年8月30日，22:10:35
td = timedelta(days=7)  # 时间间隔 = 7天
query = "TimescaleDB 函数有什么新功能？"
```

方法1：在提供的开始日期和结束日期之间进行过滤。

```python
# 方法1：查询在开始日期和结束日期之间的向量
docs_with_score = db.similarity_search_with_score(
    query, start_date=start_dt, end_date=end_dt
)
for doc, score in docs_with_score:
    print("-" * 80)
    print("得分: ", score)
    print("日期: ", doc.metadata["date"])
    print(doc.page_content)
    print("-" * 80)
```

```output
--------------------------------------------------------------------------------
得分:  0.17488396167755127
日期:  2023-08-29 18:13:24+0320
{"commit": " e4facda540286b0affba47ccc63959fefe2a7b26", "author": "Sven Klemm<sven@timescale.com>", "date": "Tue Aug 29 18:13:24 2023 +0200", "change summary": "Add compatibility layer for _timescaledb_internal functions", "change details": "With timescaledb 2.12 all the functions present in _timescaledb_internal were moved into the _timescaledb_functions schema to improve schema security. This patch adds a compatibility layer so external callers of these internal functions will not break and allow for more flexibility when migrating. "}
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分:  0.18102192878723145
日期:  2023-08-20 22:47:10+0320
{"commit": " 0a66bdb8d36a1879246bd652e4c28500c4b951ab", "author": "Sven Klemm<sven@timescale.com>", "date": "Sun Aug 20 22:47:10 2023 +0200", "change summary": "Move functions to _timescaledb_functions schema", "change details": "To increase schema security we do not want to mix our own internal objects with user objects. Since chunks are created in the _timescaledb_internal schema our internal functions should live in a different dedicated schema. This patch make the necessary adjustments for the following functions:  - to_unix_microseconds(timestamptz) - to_timestamp(bigint) - to_timestamp_without_timezone(bigint) - to_date(bigint) - to_interval(bigint) - interval_to_usec(interval) - time_to_internal(anyelement) - subtract_integer_from_now(regclass, bigint) "}
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分:  0.18150119891755445
日期:  2023-08-22 12:01:19+0320
{"commit": " cf04496e4b4237440274eb25e4e02472fc4e06fc", "author": "Sven Klemm<sven@timescale.com>", "date": "Tue Aug 22 12:01:19 2023 +0200", "change summary": "Move utility functions to _timescaledb_functions schema", "change details": "To increase schema security we do not want to mix our own internal objects with user objects. Since chunks are created in the _timescaledb_internal schema our internal functions should live in a different dedicated schema. This patch make the necessary adjustments for the following functions:  - generate_uuid() - get_git_commit() - get_os_info() - tsl_loaded() "}
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分:  0.18422493887617963
日期:  2023-08-9 15:26:03+0500
{"commit": " 44eab9cf9bef34274c88efd37a750eaa74cd8044", "author": "Konstantina Skovola<konstantina@timescale.com>", "date": "Wed Aug 9 15:26:03 2023 +0300", "change summary": "Release 2.11.2", "change details": "This release contains bug fixes since the 2.11.1 release. We recommend that you upgrade at the next available opportunity.  **Features** * #5923 Feature flags for TimescaleDB features  **Bugfixes** * #5680 Fix DISTINCT query with JOIN on multiple segmentby columns * #5774 Fixed two bugs in decompression sorted merge code * #5786 Ensure pg_config --cppflags are passed * #5906 Fix quoting owners in sql scripts. * #5912 Fix crash in 1-step integer policy creation  **Thanks** * @mrksngl for submitting a PR to fix extension upgrade scripts * @ericdevries for reporting an issue with DISTINCT queries using segmentby columns of compressed hypertable "}
--------------------------------------------------------------------------------
```

请注意，查询仅返回指定日期范围内的结果。

方法2：在提供的开始日期和时间间隔后进行筛选。

```python
# 方法2：查询开始日期和时间间隔td之间的向量
# 最相关的向量在8月1日和7天后之间
docs_with_score = db.similarity_search_with_score(
    query, start_date=start_dt, time_delta=td
)
for doc, score in docs_with_score:
    print("-" * 80)
    print("得分：", score)
    print("日期：", doc.metadata["date"])
    print(doc.page_content)
    print("-" * 80)
```

```output
--------------------------------------------------------------------------------
得分： 0.18458807468414307
日期： 2023-08-3 14:30:23+0500
{"commit": " 7aeed663b9c0f337b530fd6cad47704a51a9b2ec", "author": "Dmitry Simonenko<dmitry@timescale.com>", "date": "Thu Aug 3 14:30:23 2023 +0300", "change summary": "Feature flags for TimescaleDB features", "change details": "This PR adds several GUCs which allow to enable/disable major timescaledb features:  - enable_hypertable_create - enable_hypertable_compression - enable_cagg_create - enable_policy_create "}
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分： 0.20492422580718994
日期： 2023-08-7 18:31:40+0320
{"commit": " 07762ea4cedefc88497f0d1f8712d1515cdc5b6e", "author": "Sven Klemm<sven@timescale.com>", "date": "Mon Aug 7 18:31:40 2023 +0200", "change summary": "Test timescaledb debian 12 packages in CI", "change details": ""}
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分： 0.21106326580047607
日期： 2023-08-3 14:36:39+0500
{"commit": " 2863daf3df83c63ee36c0cf7b66c522da5b4e127", "author": "Dmitry Simonenko<dmitry@timescale.com>", "date": "Thu Aug 3 14:36:39 2023 +0300", "change summary": "Support CREATE INDEX ONLY ON main table", "change details": "This PR adds support for CREATE INDEX ONLY ON clause which allows to create index only on the main table excluding chunks.  Fix #5908 "}
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分： 0.21698051691055298
日期： 2023-08-2 20:24:14+0140
{"commit": " 3af0d282ea71d9a8f27159a6171e9516e62ec9cb", "author": "Lakshmi Narayanan Sreethar<lakshmi@timescale.com>", "date": "Wed Aug 2 20:24:14 2023 +0100", "change summary": "PG16: ExecInsertIndexTuples requires additional parameter", "change details": "PG16 adds a new boolean parameter to the ExecInsertIndexTuples function to denote if the index is a BRIN index, which is then used to determine if the index update can be skipped. The fix also removes the INDEX_ATTR_BITMAP_ALL enum value.  Adapt these changes by updating the compat function to accomodate the new parameter added to the ExecInsertIndexTuples function and using an alternative for the removed INDEX_ATTR_BITMAP_ALL enum value.  postgres/postgres@19d8e23 "}
--------------------------------------------------------------------------------
```

再次注意，我们得到了在指定时间过滤内的结果，与之前的查询不同。

方法3：在提供的结束日期和时间间隔之前进行筛选。

```python
# 方法3：查询结束日期end_dt和时间间隔td之前的向量
# 最相关的向量在8月30日和7天之前
docs_with_score = db.similarity_search_with_score(query, end_date=end_dt, time_delta=td)
for doc, score in docs_with_score:
    print("-" * 80)
    print("得分：", score)
    print("日期：", doc.metadata["date"])
    print(doc.page_content)
    print("-" * 80)
```

```output
--------------------------------------------------------------------------------
得分： 0.17488396167755127
日期： 2023-08-29 18:13:24+0320
{"commit": " e4facda540286b0affba47ccc63959fefe2a7b26", "author": "Sven Klemm<sven@timescale.com>", "date": "Tue Aug 29 18:13:24 2023 +0200", "change summary": "Add compatibility layer for _timescaledb_internal functions", "change details": "With timescaledb 2.12 all the functions present in _timescaledb_internal were moved into the _timescaledb_functions schema to improve schema security. This patch adds a compatibility layer so external callers of these internal functions will not break and allow for more flexibility when migrating. "}
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分： 0.18496227264404297
日期： 2023-08-29 10:49:47+0320
这些变更是关于将分区函数移动到_timescaledb_functions模式的。这是为了增加模式安全性，我们不希望将我们自己的内部对象与用户对象混合在一起。由于分块是在_timescaledb_internal模式中创建的，我们的内部函数应该存在于不同的专用模式中。这个补丁对以下函数进行了必要的调整：
- get_partition_for_key(val anyelement)
- get_partition_hash(val anyelement)
这些变更是关于将ddl_internal函数移动到_timescaledb_functions模式的。这是为了增加模式安全性，我们不希望将我们自己的内部对象与用户对象混合在一起。由于分块是在_timescaledb_internal模式中创建的，我们的内部函数应该存在于不同的专用模式中。这个补丁对以下函数进行了必要的调整：
- chunk_constraint_add_table_constraint(_timescaledb_catalog.chunk_constraint)
- chunk_drop_replica(regclass,name)
- chunk_index_clone(oid)
- chunk_index_replace(oid,oid)
- create_chunk_replica_table(regclass,name)
- drop_stale_chunks(name,integer[])
- health()
- hypertable_constraint_add_table_fk_constraint(name,name,name,integer)
- process_ddl_event()
- wait_subscription_sync(name,name,integer,numeric)
这些变更是关于简化模式移动更新脚本的。使用动态SQL来创建ALTER FUNCTION语句，以便为那些在先前版本中可能不存在的函数创建更新脚本。
此外，还有其他一些变更，例如将函数移动到_timescaledb_functions模式，以及为_timescaledb_internal函数添加兼容性层，以便在迁移时不会中断外部调用这些内部函数，并允许更灵活性。
以上是一些关于模式移动和函数调整的变更。这些变更旨在提高模式安全性，确保内部对象和用户对象分开，并为迁移提供更大的灵活性。
![Timescale](https://example.com/timescale_image.jpg)
```

```python
# 方法5：查询截止日期前的所有向量
docs_with_score = db.similarity_search_with_score(query, end_date=end_dt)
for doc, score in docs_with_score:
    print("-" * 80)
    print("得分：", score)
    print("日期：", doc.metadata["date"])
    print(doc.page_content)
    print("-" * 80)
```

```output
--------------------------------------------------------------------------------
得分： 0.16723191738128662
日期： 2023-04-11 22:01:14+0320
{"commit": " 0595ff0888f2ffb8d313acb0bda9642578a9ade3", "author": "Sven Klemm<sven@timescale.com>", "date": "Tue Apr 11 22:01:14 2023 +0200", "change summary": "将类型支持函数移至_timescaledb_functions模式", "change details": ""}
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分： 0.1706540584564209
日期： 2023-04-6 13:00:00+0320
{"commit": " 04f43335dea11e9c467ee558ad8edfc00c1a45ed", "author": "Sven Klemm<sven@timescale.com>", "date": "Thu Apr 6 13:00:00 2023 +0200", "change summary": "将聚合支持函数移至_timescaledb_functions模式", "change details": "此补丁将直方图、第一个和最后一个的支持函数移至_timescaledb_functions模式。由于我们在升级脚本中更改了现有函数的模式，且未更改聚合函数，因此对于使用这些聚合的任何用户对象来说，这应该完全透明。"}
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分： 0.17462033033370972
日期： 2023-03-31 08:22:57+0320
{"commit": " feef9206facc5c5f506661de4a81d96ef059b095", "author": "Sven Klemm<sven@timescale.com>", "date": "Fri Mar 31 08:22:57 2023 +0200", "change summary": "添加_timescaledb_functions模式", "change details": "当前，内部用户对象（如块和我们的函数）位于同一模式中，使得锁定该模式变得困难。此补丁添加了一个新模式_timescaledb_functions，旨在成为用于timescaledb内部函数的模式，以允许代码和块或其他用户对象的分离。"}
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分： 0.17488396167755127
日期： 2023-08-29 18:13:24+0320
{"commit": " e4facda540286b0affba47ccc63959fefe2a7b26", "author": "Sven Klemm<sven@timescale.com>", "date": "Tue Aug 29 18:13:24 2023 +0200", "change summary": "为_timescaledb_internal函数添加兼容层", "change details": "随着timescaledb 2.12的推出，_timescaledb_internal中的所有函数都已移至_timescaledb_functions模式，以提高模式安全性。此补丁添加了一个兼容层，以便这些内部函数的外部调用者不会中断，并在迁移时提供更大的灵活性。"}
--------------------------------------------------------------------------------
```

以上每个结果的主要信息是，仅返回指定时间范围内的向量。这些查询非常高效，因为它们只需要搜索相关的分区。

我们还可以将此功能用于问答，其中我们希望在指定的时间范围内找到最相关的向量，以用作回答问题的上下文。让我们看一个示例，使用Timescale Vector作为检索器：

```python
# 将时间尺度向量设置为检索器，并通过关键字参数指定开始和结束日期
retriever = db.as_retriever(search_kwargs={"start_date": start_dt, "end_date": end_dt})
```

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(temperature=0.1, model="gpt-3.5-turbo-16k")
from langchain.chains import RetrievalQA
qa_stuff = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=retriever,
    verbose=True,
)
query = (
    "TimescaleDB 函数有什么新变化？告诉我这些变化是什么时候发生的。"
)
response = qa_stuff.run(query)
print(response)
```

```output
> 进入新的 RetrievalQA 链...
> 链完成。
以下是对 TimescaleDB 函数所做的更改：
1. "为 _timescaledb_internal 函数添加兼容性层" - 此更改于 2023 年 8 月 29 日 18:13:24 +0200 完成。
2. "将函数移至 _timescaledb_functions 模式" - 此更改于 2023 年 8 月 20 日 22:47:10 +0200 完成。
3. "将实用函数移至 _timescaledb_functions 模式" - 此更改于 2023 年 8 月 22 日 12:01:19 +0200 完成。
4. "将分区函数移至 _timescaledb_functions 模式" - 此更改于 2023 年 8 月 29 日 10:49:47 +0200 完成。
```

请注意，LLM 用于撰写答案的上下文仅限于指定日期范围内的检索文档。

这显示了您如何使用时间尺度向量来增强检索增强生成，通过检索与您的查询相关的时间范围内的文档。

## 3. 使用 ANN 搜索索引加速查询

您可以通过在嵌入列上创建索引来加速相似性查询。只有在您摄入了大部分数据之后才应执行此操作。

Timescale Vector 支持以下索引：

- timescale_vector 索引 (tsv)：用于快速相似性搜索的磁盘-ANN 风格图形索引（默认）。

- pgvector 的 HNSW 索引：用于快速相似性搜索的分层可导航小世界图形索引。

- pgvector 的 IVFFLAT 索引：用于快速相似性搜索的倒排文件索引。

重要提示：在 PostgreSQL 中，每个表只能在特定列上有一个索引。因此，如果您想测试不同索引类型的性能，可以通过 (1) 创建具有不同索引的多个表，(2) 在同一表中创建多个向量列并在每个列上创建不同的索引，或者 (3) 通过删除并重新创建同一列上的索引并比较结果来执行测试。

```python
# 初始化现有的 TimescaleVector 存储
COLLECTION_NAME = "timescale_commits"
embeddings = OpenAIEmbeddings()
db = TimescaleVector(
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
    embedding_function=embeddings,
)
```

使用 `create_index()` 函数而不带额外参数将默认创建 timescale_vector_index。

```python
# 创建一个索引
# 默认情况下，这将创建一个 Timescale Vector (DiskANN) 索引
db.create_index()
```

您还可以为索引指定参数。有关不同参数及其对性能的影响的全面讨论，请参阅 Timescale Vector 文档。

注意：您不需要指定参数，因为我们设置了智能默认值。但是，如果您想为特定数据集实验并获得更多性能，可以始终指定自己的参数。

```python
# 删除旧索引
db.drop_index()
# 创建一个索引
# 注意：您不需要指定 m 和 ef_construction 参数，因为我们设置了智能默认值。
db.create_index(index_type="tsv", max_alpha=1.0, num_neighbors=50)
```

Timescale Vector 还支持 HNSW ANN 索引算法，以及 ivfflat ANN 索引算法。只需在 `index_type` 参数中指定您想要创建的索引，并可选择为索引指定参数。

```python
# 删除旧索引
db.drop_index()
# 创建一个 HNSW 索引
# 注意：您不需要指定 m 和 ef_construction 参数，因为我们设置了智能默认值。
db.create_index(index_type="hnsw", m=16, ef_construction=64)
```

```python
# 删除旧索引
db.drop_index()
# 创建一个 IVFFLAT 索引
# 注意：您不需要指定 num_lists 和 num_records 参数，因为我们设置了智能默认值。
db.create_index(index_type="ivfflat", num_lists=20, num_records=1000)
```

一般来说，我们建议使用默认的 timescale vector 索引，或者 HNSW 索引。

```python
# 删除旧索引
db.drop_index()
# 创建一个新的 timescale vector 索引
db.create_index()
```

## 4. 使用时间尺度向量进行自查询检索

Timescale Vector 也支持自查询检索功能，这使其能够对自身进行查询。给定一个带有查询语句和过滤器（单个或复合）的自然语言查询，检索器使用查询构造 LLM 链来编写 SQL 查询，然后将其应用于 Timescale Vector 的底层 PostgreSQL 数据库中的向量存储。

有关自查询的更多信息，请参阅[文档](/docs/how_to/self_query)。

为了说明 Timescale Vector 的自查询功能，我们将使用第三部分中的相同的 gitlog 数据集。

```python
COLLECTION_NAME = "timescale_commits"
vectorstore = TimescaleVector(
    embedding_function=OpenAIEmbeddings(),
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
)
```

接下来，我们将创建自查询检索器。为此，我们需要提供关于文档支持的元数据字段的一些信息以及文档内容的简短描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import OpenAI
# 为元数据字段提供 LLM 信息
metadata_field_info = [
    AttributeInfo(
        name="id",
        description="从提交日期生成的 UUID v1",
        type="uuid",
    ),
    AttributeInfo(
        name="date",
        description="提交日期（timestamptz 格式）",
        type="timestamptz",
    ),
    AttributeInfo(
        name="author_name",
        description="提交者的姓名",
        type="string",
    ),
    AttributeInfo(
        name="author_email",
        description="提交者的电子邮件地址",
        type="string",
    ),
]
document_content_description = "包含提交哈希、作者、提交日期、更改摘要和更改详细信息的 git 日志提交摘要"
# 从 LLM 实例化自查询检索器
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description,
    metadata_field_info,
    enable_limit=True,
    verbose=True,
)
```

现在让我们在我们的 gitlog 数据集上测试自查询检索器。

运行下面的查询，并注意您可以用自然语言指定查询、带有过滤器的查询以及带有复合过滤器的查询（AND、OR 过滤器），自查询检索器将把该查询转换为 SQL 并在 Timescale Vector PostgreSQL 向量存储上执行搜索。

这说明了自查询检索器的强大功能。您可以使用它在向量存储上执行复杂的搜索，而无需您或您的用户直接编写任何 SQL！

```python
# 此示例指定了一个相关查询
retriever.invoke("对连续聚合进行了哪些改进？")
```

```output
/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/libs/langchain/langchain/chains/llm.py:275: UserWarning: The predict_and_parse method is deprecated, instead pass an output parser directly to LLMChain.
  warnings.warn(
```

```output
query='improvements to continuous aggregates' filter=None limit=None
```

```output
[Document(page_content='{"commit": " 35c91204987ccb0161d745af1a39b7eb91bc65a5", "author": "Fabr\\u00edzio de Royes Mello<fabriziomello@gmail.com>", "date": "Thu Nov 24 13:19:36 2022 -0300", "change summary": "Add Hierarchical Continuous Aggregates validations", "change details": "Commit 3749953e introduce Hierarchical Continuous Aggregates (aka Continuous Aggregate on top of another Continuous Aggregate) but it lacks of some basic validations.  Validations added during the creation of a Hierarchical Continuous Aggregate:  * Forbid create a continuous aggregate with fixed-width bucket on top of   a continuous aggregate with variable-width bucket.  * Forbid incompatible bucket widths:   - should not be equal;   - bucket width of the new continuous aggregate should be greater than     the source continuous aggregate;   - bucket width of the new continuous aggregate should be multiple of     the source continuous aggregate. "}', metadata={'id': 'c98d1c00-6c13-11ed-9bbe-23925ce74d13', 'date': '2022-11-24 13:19:36+-500', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 446, 'author_name': 'Fabrízio de Royes Mello', 'commit_hash': ' 35c91204987ccb0161d745af1a39b7eb91bc65a5', 'author_email': 'fabriziomello@gmail.com'}),
 Document(page_content='{"commit": " 3749953e9704e45df8f621607989ada0714ce28d", "author": "Fabr\\u00edzio de Royes Mello<fabriziomello@gmail.com>", "date": "Wed Oct 5 18:45:40 2022 -0300", "change summary": "Hierarchical Continuous Aggregates", "change details": "Enable users create Hierarchical Continuous Aggregates (aka Continuous Aggregates on top of another Continuous Aggregates).  With this PR users can create levels of aggregation granularity in Continuous Aggregates making the refresh process even faster.  A problem with this feature can be in upper levels we can end up with the \\"average of averages\\". But to get the \\"real average\\" we can rely on \\"stats_aggs\\" TimescaleDB Toolkit function that calculate and store the partials that can be finalized with other toolkit functions like \\"average\\" and \\"sum\\".  Closes #1400 "}', metadata={'id': '0df31a00-44f7-11ed-9794-ebcc1227340f', 'date': '2022-10-5 18:45:40+-500', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 470, 'author_name': 'Fabrízio de Royes Mello', 'commit_hash': ' 3749953e9704e45df8f621607989ada0714ce28d', 'author_email': 'fabriziomello@gmail.com'}),
 Document(page_content='{"commit": " a6ff7ba6cc15b280a275e5acd315741ec9c86acc", "author": "Mats Kindahl<mats@timescale.com>", "date": "Tue Feb 28 12:04:17 2023 +0100", "change summary": "Rename columns in old-style continuous aggregates", "change details": "For continuous aggregates with the old-style partial aggregates renaming columns that are not in the group-by clause will generate an error when upgrading to a later version. The reason is that it is implicitly assumed that the name of the column is the same as for the direct view. This holds true for new-style continous aggregates, but is not always true for old-style continuous aggregates. In particular, columns that are not part of the `GROUP BY` clause can have an internally generated name.  This commit fixes that by extracting the name of the column from the partial view and use that when renaming the partial view column and the materialized table column. "}', metadata={'id': 'a49ace80-b757-11ed-8138-2390fd44ffd9', 'date': '2023-02-28 12:04:17+0140', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 294, 'author_name': 'Mats Kindahl', 'commit_hash': ' a6ff7ba6cc15b280a275e5acd315741ec9c86acc', 'author_email': 'mats@timescale.com'}),
 Document(page_content='{"commit": " 5bba74a2ec083728f8e93e09d03d102568fd72b5", "author": "Fabr\\u00edzio de Royes Mello<fabriziomello@gmail.com>", "date": "Mon Aug 7 19:49:47 2023 -0300", "change summary": "Relax strong table lock when refreshing a CAGG", "change details": "When refreshing a Continuous Aggregate we take a table lock on _timescaledb_catalog.continuous_aggs_invalidation_threshold when processing the invalidation logs (the first transaction of the refresh Continuous Aggregate procedure). It means that even two different Continuous Aggregates over two different hypertables will wait each other in the first phase of the refreshing procedure. Also it lead to problems when a pg_dump is running because it take an AccessShareLock on tables so Continuous Aggregate refresh execution will wait until the pg_dump finish.  Improved it by relaxing the strong table-level lock to a row-level lock so now the Continuous Aggregate refresh procedure can be executed in multiple sessions with less locks.  Fix #3554 "}', metadata={'id': 'b5583780-3574-11ee-a5ba-2e305874a58f', 'date': '2023-08-7 19:49:47+-500', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 27, 'author_name': 'Fabrízio de Royes Mello', 'commit_hash': ' 5bba74a2ec083728f8e93e09d03d102568fd72b5', 'author_email': 'fabriziomello@gmail.com'})]
```

```python
# 这个例子指定了一个过滤器
retriever.invoke("Sven Klemm 添加了哪些提交？")
```

```output
query=' ' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='author_name', value='Sven Klemm') limit=None
```

```output
[Document(page_content='{"commit": " e2e7ae304521b74ac6b3f157a207da047d44ab06", "author": "Sven Klemm<sven@timescale.com>", "date": "Fri Mar 3 11:22:06 2023 +0100", "change summary": "Don\'t run sanitizer test on individual PRs", "change details": "Sanitizer tests take a long time to run so we don\'t want to run them on individual PRs but instead run them nightly and on commits to master. "}', metadata={'id': '3f401b00-b9ad-11ed-b5ea-a3fd40b9ac16', 'date': '2023-03-3 11:22:06+0140', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 295, 'author_name': 'Sven Klemm', 'commit_hash': ' e2e7ae304521b74ac6b3f157a207da047d44ab06', 'author_email': 'sven@timescale.com'}),
 Document(page_content='{"commit": " d8f19e57a04d17593df5f2c694eae8775faddbc7", "author": "Sven Klemm<sven@timescale.com>", "date": "Wed Feb 1 08:34:20 2023 +0100", "change summary": "Bump version of setup-wsl github action", "change details": "The currently used version pulls in Node.js 12 which is deprecated on github.  https://github.blog/changelog/2022-09-22-github-actions-all-actions-will-begin-running-on-node16-instead-of-node12/ "}', metadata={'id': 'd70de600-a202-11ed-85d6-30b6df240f49', 'date': '2023-02-1 08:34:20+0140', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 350, 'author_name': 'Sven Klemm', 'commit_hash': ' d8f19e57a04d17593df5f2c694eae8775faddbc7', 'author_email': 'sven@timescale.com'}),
 Document(page_content='{"commit": " 83b13cf6f73a74656dde9cc6ec6cf76740cddd3c", "author": "Sven Klemm<sven@timescale.com>", "date": "Fri Nov 25 08:27:45 2022 +0100", "change summary": "Use packaged postgres for sqlsmith and coverity CI", "change details": "The sqlsmith and coverity workflows used the cache postgres build but could not produce a build by themselves and therefore relied on other workflows to produce the cached binaries. This patch changes those workflows to use normal postgres packages instead of custom built postgres to remove that dependency. "}', metadata={'id': 'a786ae80-6c92-11ed-bd6c-a57bd3348b97', 'date': '2022-11-25 08:27:45+0140', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 447, 'author_name': 'Sven Klemm', 'commit_hash': ' 83b13cf6f73a74656dde9cc6ec6cf76740cddd3c', 'author_email': 'sven@timescale.com'}),
 Document(page_content='{"commit": " b1314e63f2ff6151ab5becfb105afa3682286a4d", "author": "Sven Klemm<sven@timescale.com>", "date": "Thu Dec 22 12:03:35 2022 +0100", "change summary": "Fix RPM package test for PG15 on centos 7", "change details": "Installing PG15 on Centos 7 requires the EPEL repository to satisfy the dependencies. "}', metadata={'id': '477b1d80-81e8-11ed-9c8c-9b5abbd67c98', 'date': '2022-12-22 12:03:35+0140', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 408, 'author_name': 'Sven Klemm', 'commit_hash': ' b1314e63f2ff6151ab5becfb105afa3682286a4d', 'author_email': 'sven@timescale.com'})]
```

```python
# 这个例子指定了一个查询和过滤器
retriever.invoke("Sven Klemm 添加了哪些关于 timescaledb_functions 的提交？")
```

```output
query='timescaledb_functions' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='author_name', value='Sven Klemm') limit=None
```

```output
[Document(page_content='{"commit": " 04f43335dea11e9c467ee558ad8edfc00c1a45ed", "author": "Sven Klemm<sven@timescale.com>", "date": "Thu Apr 6 13:00:00 2023 +0200", "change summary": "Move aggregate support function into _timescaledb_functions", "change details": "This patch moves the support functions for histogram, first and last into the _timescaledb_functions schema. Since we alter the schema of the existing functions in upgrade scripts and do not change the aggregates this should work completely transparently for any user objects using those aggregates. "}', metadata={'id': '2cb47800-d46a-11ed-8f0e-2b624245c561', 'date': '2023-04-6 13:00:00+0320', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 233, 'author_name': 'Sven Klemm', 'commit_hash': ' 04f43335dea11e9c467ee558ad8edfc00c1a45ed', 'author_email': 'sven@timescale.com'}),
```

```markdown
# 2023 年 3 月 31 日提交
- 提交者：Sven Klemm
- 提交哈希：feef9206facc5c5f506661de4a81d96ef059b095
- 变更摘要：添加 _timescaledb_functions 模式
- 变更详情：目前，内部用户对象（如块和我们的函数）存储在同一个模式中，使得难以锁定该模式。此补丁添加了一个新的模式 _timescaledb_functions，用于存储 timescaledb 内部函数，以实现代码和块或其他用户对象的分离。 
# 2023 年 8 月 20 日提交
- 提交者：Sven Klemm
- 提交哈希：0a66bdb8d36a1879246bd652e4c28500c4b951ab
- 变更摘要：将函数移动到 _timescaledb_functions 模式
- 变更详情：为了增加模式安全性，我们不希望将我们自己的内部对象与用户对象混合在一起。由于块是在 _timescaledb_internal 模式中创建的，我们的内部函数应该存储在一个不同的专用模式中。此补丁对以下函数进行了必要的调整：- to_unix_microseconds(timestamptz) - to_timestamp(bigint) - to_timestamp_without_timezone(bigint) - to_date(bigint) - to_interval(bigint) - interval_to_usec(interval) - time_to_internal(anyelement) - subtract_integer_from_now(regclass, bigint)
# 2023 年 4 月 13 日提交
- 提交者：Sven Klemm
- 提交哈希：56ea8b4de93cefc38e002202d8ac96947dcbaa77
- 变更摘要：将触发器函数移动到 _timescaledb_functions 模式
- 变更详情：为了增加模式安全性，我们不希望将我们自己的内部对象与用户对象混合在一起。由于块是在 _timescaledb_internal 模式中创建的，我们的内部函数应该存储在一个不同的专用模式中。此补丁对我们的触发器函数进行了必要的调整。
## 查询结果
- 提交哈希：5cf354e2469ee7e43248bed382a4b49fc7ccfecd
- 提交者：Markus Engel
- 提交日期：2023 年 7 月 31 日
- 变更摘要：修复 SQL 脚本中的引号所有者
- 变更详情：在从字符串类型引用角色时，必须在将其转换为 regrole 之前使用 pg_catalog.quote_ident 正确引用。修复了这一点，特别是在更新脚本中。
```

```markdown
# 允许在 CAGGs 上进行副本标识（修改表）(#5868)
这个提交是 #5515 的后续，该提交增加了对 hypertable 上 ALTER TABLE REPLICA IDENTITY (FULL | INDEX) 的支持。
这个提交允许对物化 hypertable 执行，以便在为其启用逻辑复制时，对连续聚合进行更新/删除操作。
作者：noctarius aka Christoph Engelbert
提交日期：2023年7月12日
[20]
---
# 修复 PG12 移除后的破损 CI
提交 cdea343cc 更新了 gh_matrix_builder.py 脚本，但未将 PG_LATEST 变量导入脚本，从而破坏了 CI。导入该变量以修复 CI 测试。
作者：Lakshmi Narayanan Sreethar
提交日期：2023年7月25日
[21]
---
# 修复 SQLSmith 工作流
构建失败是因为选择了错误版本的 Postgres。移除它。
作者：Alexander Kuzmenkov
提交日期：2023年7月28日
[22]
---
# 添加分层连续聚合验证
提交 3749953e 引入了分层连续聚合（也称为另一个连续聚合的顶部连续聚合），但缺少一些基本验证。在创建分层连续聚合期间添加的验证：
* 禁止在具有可变宽度桶的连续聚合的顶部创建具有固定宽度桶的连续聚合。
* 禁止不兼容的桶宽度：
  - 不应该相等；
  - 新连续聚合的桶宽度应大于源连续聚合；
  - 新连续聚合的桶宽度应是源连续聚合的倍数。
作者：Fabrízio de Royes Mello
提交日期：2022年11月24日
[23]
```

## 5. 使用现有的 TimescaleVector 向量存储库

在上面的示例中，我们从一组文档创建了一个向量存储库。然而，通常我们希望向现有的向量存储库中插入数据并查询数据。让我们看看如何初始化、向现有的 TimescaleVector 向量存储库添加文档并进行查询。

要使用现有的 Timescale Vector 存储库，我们需要知道要查询的表的名称（`COLLECTION_NAME`）和云 PostgreSQL 数据库的 URL（`SERVICE_URL`）。

```python
# 初始化现有的
COLLECTION_NAME = "timescale_commits"
embeddings = OpenAIEmbeddings()
vectorstore = TimescaleVector(
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
    embedding_function=embeddings,
)
```

要将新数据加载到表中，我们使用 `add_document()` 函数。该函数接受一个文档列表和一个元数据列表。元数据必须包含每个文档的唯一 id。

如果您希望您的文档与当前日期和时间相关联，您不需要创建 id 列表。uuid 将自动生成用于每个文档。

如果您希望您的文档与过去的日期和时间相关联，您可以使用 `timecale-vector` Python 库中的 `uuid_from_time` 函数创建 id 列表，如上面第 2 节所示。该函数接受一个 datetime 对象，并返回一个带有日期和时间编码的 uuid。

```python
# 向 TimescaleVector 中的集合添加文档
ids = vectorstore.add_documents([Document(page_content="foo")])
ids
```

```output
['a34f2b8a-53d7-11ee-8cc3-de1e4b2a0118']
```

```python
# 查询相似文档的向量存储库
docs_with_score = vectorstore.similarity_search_with_score("foo")
```

```python
docs_with_score[0]
```

```output
(Document(page_content='foo', metadata={}), 5.006789860928507e-06)
```

```python
docs_with_score[1]
```

```output
(Document(page_content='{"commit": " 00b566dfe478c11134bcf1e7bcf38943e7fafe8f", "author": "Fabr\\u00edzio de Royes Mello<fabriziomello@gmail.com>", "date": "Mon Mar 6 15:51:03 2023 -0300", "change summary": "Remove unused functions", "change details": "We don\'t use `ts_catalog_delete[_only]` functions anywhere and instead we rely on `ts_catalog_delete_tid[_only]` functions so removing it from our code base. "}', metadata={'id': 'd7f5c580-bc4f-11ed-9712-ffa0126a201a', 'date': '2023-03-6 15:51:03+-500', 'source': '/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/docs/docs/modules/ts_git_log.json', 'seq_num': 285, 'author_name': 'Fabrízio de Royes Mello', 'commit_hash': ' 00b566dfe478c11134bcf1e7bcf38943e7fafe8f', 'author_email': 'fabriziomello@gmail.com'}), 0.23607668446580354)
```

### 删除数据

您可以通过 uuid 或元数据筛选来删除数据。

```python
ids = vectorstore.add_documents([Document(page_content="Bar")])
vectorstore.delete(ids)
```

```output
True
```

使用元数据进行删除在您想要定期更新从特定来源获取的信息，或者特定日期或其他元数据属性时特别有用。

```python
vectorstore.add_documents(
    [Document(page_content="Hello World", metadata={"source": "www.example.com/hello"})]
)
vectorstore.add_documents(
    [Document(page_content="Adios", metadata={"source": "www.example.com/adios"})]
)
vectorstore.delete_by_metadata({"source": "www.example.com/adios"})
vectorstore.add_documents(
    [
        Document(
            page_content="Adios, but newer!",
            metadata={"source": "www.example.com/adios"},
        )
    ]
```

### 覆盖向量存储

如果您有一个现有的集合，可以通过执行 `from_documents` 并设置 `pre_delete_collection` = True 来覆盖它。

```python
db = TimescaleVector.from_documents(
    documents=docs,
    embedding=embeddings,
    collection_name=COLLECTION_NAME,
    service_url=SERVICE_URL,
    pre_delete_collection=True,
)
```

```python
docs_with_score = db.similarity_search_with_score("foo")
```

```python
docs_with_score[0]
```