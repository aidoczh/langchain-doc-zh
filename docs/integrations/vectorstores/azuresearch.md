# Azure AI 搜索

[Azure AI Search](https://learn.microsoft.com/azure/search/search-what-is-azure-search)（之前称为 `Azure Search` 和 `Azure Cognitive Search`）是一项云搜索服务，为开发人员提供基础设施、API 和工具，以便在规模上进行向量、关键字和混合查询的信息检索。

## 安装 Azure AI Search SDK

使用 azure-search-documents 包的 11.4.0 版本或更高版本。

```python
%pip install --upgrade --quiet  azure-search-documents
%pip install --upgrade --quiet  azure-identity
```

## 导入所需的库

假设已导入 `OpenAIEmbeddings`，但如果使用 Azure OpenAI，则应导入 `AzureOpenAIEmbeddings`。

```python
import os
from langchain_community.vectorstores.azuresearch import AzureSearch
from langchain_openai import AzureOpenAIEmbeddings, OpenAIEmbeddings
```

## 配置 OpenAI 设置

设置 OpenAI 提供商的变量。您需要拥有 [OpenAI 账户](https://platform.openai.com/docs/quickstart?context=python) 或 [Azure OpenAI 账户](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/create-resource) 来生成嵌入。

```python
# 选项 1：使用 OpenAI 账户
openai_api_key: str = "YOUR_API_KEY"
openai_api_version: str = "2023-05-15"
model: str = "text-embedding-ada-002"
```
```python
# 选项 2：使用 Azure OpenAI 账户部署嵌入模型
azure_endpoint: str = "YOUR_AZURE_OPENAI_ENDPOINT"
azure_openai_api_key: str = "YOUR_AZURE_OPENAI_KEY"
azure_openai_api_version: str = "2023-05-15"
azure_deployment: str = "text-embedding-ada-002"
```

## 配置向量存储设置

您需要一个 [Azure 订阅](https://azure.microsoft.com/en-us/free/search) 和 [Azure AI Search 服务](https://learn.microsoft.com/azure/search/search-create-service-portal) 来使用此向量存储集成。小型和有限工作负载的免费版本可用。

设置 Azure AI Search URL 和管理员 API 密钥的变量。您可以从 [Azure 门户](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 获取这些变量。

```python
vector_store_address: str = "YOUR_AZURE_SEARCH_ENDPOINT"
vector_store_password: str = "YOUR_AZURE_SEARCH_ADMIN_KEY"
```

## 创建嵌入和向量存储实例

创建 OpenAIEmbeddings 和 AzureSearch 类的实例。完成此步骤后，您应该在 Azure AI Search 资源上有一个空的搜索索引。集成模块提供了默认模式。

```python
# 选项 1：使用 OpenAI 账户的 OpenAIEmbeddings
embeddings: OpenAIEmbeddings = OpenAIEmbeddings(
    openai_api_key=openai_api_key, openai_api_version=openai_api_version, model=model
)
```
```python
# 选项 2：使用 Azure 账户的 AzureOpenAIEmbeddings
embeddings: AzureOpenAIEmbeddings = AzureOpenAIEmbeddings(
    azure_deployment=azure_deployment,
    openai_api_version=azure_openai_api_version,
    azure_endpoint=azure_endpoint,
    api_key=azure_openai_api_key,
)
```

## 创建向量存储实例

使用上述嵌入创建 AzureSearch 类的实例。

```python
index_name: str = "langchain-vector-demo"
vector_store: AzureSearch = AzureSearch(
    azure_search_endpoint=vector_store_address,
    azure_search_key=vector_store_password,
    index_name=index_name,
    embedding_function=embeddings.embed_query,
)
```

## 将文本和嵌入插入向量存储

此步骤加载、分块和向量化示例文档，然后将内容索引到 Azure AI Search 上的搜索索引中。

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt", encoding="utf-8")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
vector_store.add_documents(documents=docs)
```
```output
['M2U1OGM4YzAtYjMxYS00Nzk5LTlhNDgtZTc3MGVkNTg1Mjc0',
 'N2I2MGNiZDEtNDdmZS00YWNiLWJhYTYtYWEzMmFiYzU1ZjZm',
 'YWFmNDViNTQtZTc4MS00MTdjLTkzZjQtYTJkNmY1MDU4Yzll',
 'MjgwY2ExZDctYTUxYi00NjE4LTkxMjctZDA1NDQ1MzU4NmY1',
 'NGE4NzhkNTAtZWYxOC00ZmI5LTg0MTItZDQ1NzMxMWVmMTIz',
 'MTYwMWU3YjAtZDIzOC00NTYwLTgwMmEtNDI1NzA2MWVhMDYz',
 'NGM5N2NlZjgtMTc5Ny00OGEzLWI5YTgtNDFiZWE2MjBlMzA0',
 'OWQ4M2MyMTYtMmRkNi00ZDUxLWI0MDktOGE2NjMxNDFhYzFm',
 'YWZmZGJkOTAtOGM3My00MmNiLTg5OWUtZGMwMDQwYTk1N2Vj',
 'YTc3MTI2OTktYmVkMi00ZGU4LTgyNmUtNTY1YzZjMDg2YWI3',
 'MTQwMmVlYjEtNDI0MS00N2E0LWEyN2ItZjhhYWU0YjllMjRk',
 'NjJjYWY4ZjctMzgyNi00Y2I5LTkwY2UtZjRkMjJhNDQxYTFk',
 'M2ZiM2NiYTMtM2ZiMS00YWJkLWE3ZmQtNDZiODcyOTMyYWYx',
 'MzNmZTNkMWYtMjNmYS00Y2NmLTg3ZjQtYTZjOWM1YmJhZTRk',
 'ZDY3MDc1NzYtY2YzZS00ZjExLWEyMjAtODhiYTRmNDUzMTBi',
 'ZGIyYzA4NzUtZGM2Ni00MDUwLWEzZjYtNTg3MDYyOWQ5MWQy',
 'NTA0MjBhMzYtOTYzMi00MDQ2LWExYWQtMzNiN2I4ODM4ZGZl',
 'OTdjYzU2NGUtNWZjNC00N2ZmLWExMjQtNjhkYmZkODg4MTY3',
 'OThhMWZmMjgtM2EzYS00OWZkLTk1NGEtZTdkNmRjNWYxYmVh',
 'ZGVjMTQ0NzctNDVmZC00ZWY4LTg4N2EtMDQ1NWYxNWM5NDVh',
 'MjRlYzE4YzItZTMxNy00OGY3LThmM2YtMjM0YmRhYTVmOGY3',
 'MWU0NDA3ZDQtZDE4MS00OWMyLTlmMzktZjdkYzZhZmUwYWM3',
 'ZGM2ZDhhY2MtM2NkNi00MzZhLWJmNTEtMmYzNjEwMzE3NmZl',
 'YjBmMjkyZTItYTNlZC00MmY2LThiMzYtMmUxY2MyNDlhNGUw',
 'OThmYTQ0YzEtNjk0MC00NWIyLWE1ZDQtNTI2MTZjN2NlODcw',
 'NDdlOGU1ZGQtZTVkMi00M2MyLWExN2YtOTc2ODk3OWJmNmQw',
 'MDVmZGNkYTUtNWI2OS00YjllLTk0YTItZDRmNWQxMWU3OTVj',
 'YWFlNTVmNjMtMDZlNy00NmE5LWI0ODUtZTI3ZTFmZWRmNzU0',
 'MmIzOTkxODQtODYxMi00YWM2LWFjY2YtNjRmMmEyM2JlNzMw',
 'ZmI1NDhhNWItZWY0ZS00NTNhLWEyNDEtMTE2OWYyMjc4YTU2',
 'YTllYTc5OTgtMzJiNC00ZjZjLWJiMzUtNWVhYzFjYzgxMjU2',
 'ODZlZWUyOTctOGY4OS00ZjA3LWIyYTUtNDVlNDUyN2E4ZDFk',
 'Y2M0MWRlM2YtZDU4Ny00MjZkLWE5NzgtZmRkMTNhZDg2YjEy',
 'MDNjZWQ2ODEtMWZiMy00OTZjLTk3MzAtZjE4YjIzNWVhNTE1',
 'OTE1NDY0NzMtODNkZS00MTk4LTk4NWQtZGVmYjQ2YjFlY2Q0',
 'ZTgwYWQwMjEtN2ZlOS00NDk2LWIxNzUtNjk2ODE3N2U0Yzlj',
 'ZDkxOTgzMGUtZGExMC00Yzg0LWJjMGItOWQ2ZmUwNWUwOGJj',
 'ZGViMGI2NDEtZDdlNC00YjhiLTk0MDUtYjEyOTVlMGU1Y2I2',
 'ODliZTYzZTctZjdlZS00YjBjLWFiZmYtMDJmNjQ0YjU3ZDcy',
 'MDFjZGI1NzUtOTc0Ni00NWNmLThhYzYtYzRlZThkZjMwM2Vl',
 'ZjY2ZmRiN2EtZWVhNS00ODViLTk4YjYtYjQ2Zjc4MDdkYjhk',
 'ZTQ3NDMwODEtMTQwMy00NDFkLWJhZDQtM2UxN2RkOTU1MTdl']
```

## 执行向量相似性搜索

使用 similarity_search() 方法执行纯向量相似性搜索：

```python
# 执行相似性搜索
docs = vector_store.similarity_search(
    query="总统对Ketanji Brown Jackson说了什么",
    k=3,
    search_type="similarity",
)
print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一就是提名某人担任美国最高法院的法官。
而我在4天前就做到了，当我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶司法部长的卓越传统。
```

## 执行带相关性分数的向量相似性搜索

使用 similarity_search_with_relevance_scores() 方法执行纯向量相似性搜索。不符合阈值要求的查询将被排除。

```python
docs_and_scores = vector_store.similarity_search_with_relevance_scores(
    query="总统对Ketanji Brown Jackson说了什么",
    k=4,
    score_threshold=0.80,
)
from pprint import pprint
pprint(docs_and_scores)
```
```output
[(Document(page_content='今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。\n\n总统最严肃的宪法责任之一就是提名某人担任美国最高法院的法官。\n\n而我在4天前就做到了，当我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶司法部长的卓越传统。', metadata={'source': '../../how_to/state_of_the_union.txt'}),
  0.84402436),
 (Document(page_content='一个曾在私人执业中担任高级诉讼律师。一个前联邦公共辩护人。来自公立学校教育工作者和警察家庭。一个达成共识的人。自她被提名以来，她得到了广泛的支持——从警察兄弟会到民主党和共和党任命的前法官。\n\n如果我们要推进自由和正义，我们需要保护边境并修复移民制度。\n\n我们可以做到。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测走私毒品。\n\n我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。\n\n我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭能够更快地得到审理。\n\n我们正在承诺并支持南美和中美的伙伴，以接纳更多的难民并保护他们自己的边界。', metadata={'source': '../../how_to/state_of_the_union.txt'}),
  0.82128483),
 (Document(page_content='对于我们的 LGBTQ+ 美国人，让我们最终将两党支持的《平等法案》送到我的办公桌上。针对跨性别美国人及其家人的一系列州法律的袭击是错误的。\n\n正如我去年所说，特别是对我们年轻的跨性别美国人，作为你们的总统，我将永远支持你们，这样你们就可以做自己，并发挥上帝赋予你们的潜力。\n\n虽然我们经常看起来似乎永远不会达成一致，但这并不是真的。去年，我签署了80项两党法案。从防止政府关门到保护亚裔免受仍然普遍存在的仇恨犯罪，再到改革军事司法。\n\n很快，我们将加强我三十年前首次起草的《反对妇女暴力法案》。对我们来说，向国家展示我们可以团结一致并做出重大成就是很重要的。\n\n因此，今晚我提出了一个国家的团结议程。我们可以一起做的四件大事。\n\n首先，战胜阿片类药物流行病。', metadata={'source': '../../how_to/state_of_the_union.txt'}),
  0.8151042)]
今晚，我宣布要打击那些向美国企业和消费者收取过高费用的公司。
随着华尔街公司接管更多的养老院，这些养老院的质量下降，成本上升。这种情况将在我的任期内结束。医疗保险将为养老院设定更高的标准，确保您所关爱的人得到他们应得和期望的照顾。
我们还将通过给予工人公平机会、提供更多培训和学徒计划、根据技能而非学位来雇佣他们，来降低成本，保持经济强劲增长。让我们通过通过《工资公平法案》和带薪休假来实现这一目标。将最低工资提高到每小时15美元，并延长儿童税收抵免期限，这样就不会有人在贫困中抚养家庭。
让我们增加Pell Grants的发放额度，增加我们对历史悠久的黑人大学和学院的支持，并投资于吉尔所称的美国最佳保密的秘密：社区学院。 [20]
---
## 执行混合搜索
使用`search_type`或`hybrid_search()`方法执行混合搜索。向量和非向量文本字段同时进行查询，结果合并，返回统一结果集的前几个匹配项。
```python
# 使用`search_type`参数执行混合搜索
docs = vector_store.similarity_search(
    query="总统对Ketanji Brown Jackson有何评论",
    k=3,
    search_type="hybrid",
)
print(docs[0].page_content)
```
```output

今晚，我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便再通过《揭示法案》，这样美国人就可以知道谁在资助我们的选举。

今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶法官，感谢您的服务。

总统最严肃的宪法责任之一是提名某人担任美国最高法院大法官。

而我在4天前就做到了，当时我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智囊，将继续延续布雷耶法官的卓越传统。

```
```python
# 使用`hybrid_search`方法执行混合搜索
docs = vector_store.hybrid_search(
    query="总统对Ketanji Brown Jackson有何评论", k=3
)
print(docs[0].page_content)
```
```output

今晚，我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便再通过《揭示法案》，这样美国人就可以知道谁在资助我们的选举。

今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶法官，感谢您的服务。

总统最严肃的宪法责任之一是提名某人担任美国最高法院大法官。

而我在4天前就做到了，当时我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智囊，将继续延续布雷耶法官的卓越传统。

```
---
## 自定义模式和查询
本节将向您展示如何使用自定义模式替换默认模式。
### 使用自定义可过滤字段创建新索引
此模式显示字段定义。它是默认模式，还包括几个新字段，被标记为可过滤。由于它使用默认的向量配置，您在这里看不到向量配置或向量配置覆盖。默认向量配置的名称是“myHnswProfile”，它使用Hierarchical Navigable Small World (HNSW)进行针对`content_vector`字段的索引和查询。
在此步骤中，此模式没有数据。当您执行该单元格时，您应该在Azure AI Search上获得一个空索引。
```python
from azure.search.documents.indexes.models import (
    ScoringProfile,
    SearchableField,
    SearchField,
    SearchFieldDataType,
    SimpleField,
    TextWeights,
)
# 如果Azure OpenAI是您的提供商，请将OpenAIEmbeddings替换为AzureOpenAIEmbeddings。
embeddings: OpenAIEmbeddings = OpenAIEmbeddings(
    openai_api_key=openai_api_key, openai_api_version=openai_api_version, model=model
)
embedding_function = embeddings.embed_query
fields = [
    SimpleField(
        name="id",
        type=SearchFieldDataType.String,
        key=True,
        filterable=True,
    ),
    SearchableField(
        name="content",
        type=SearchFieldDataType.String,
        searchable=True,
    ),
    SearchField(
        name="content_vector",
        type=SearchFieldDataType.Collection(SearchFieldDataType.Single),
        searchable=True,
```
```python
from azure.search.documents.indexes.models import (
    FreshnessScoringFunction,
    FreshnessScoringParameters,
    ScoringProfile,
    SearchableField,
    SearchField,
    SearchFieldDataType,
    SimpleField,
    TextWeights,
)
# 如果 Azure OpenAI 是您的提供商，请将 OpenAIEmbeddings 替换为 AzureOpenAIEmbeddings。
embeddings: OpenAIEmbeddings = OpenAIEmbeddings(
    openai_api_key=openai_api_key, openai_api_version=openai_api_version, model=model
)
embedding_function = embeddings.embed_query
fields = [
    SimpleField(
        name="id",
        type=SearchFieldDataType.String,
        key=True,
        filterable=True,
    ),
    SearchableField(
        name="content",
        type=SearchFieldDataType.String,
        searchable=True,
    ),
    SearchField(
        name="content_vector",
        type=SearchFieldDataType.Collection(SearchFieldDataType.Single),
        searchable=True,
        vector_search_dimensions=len(embedding_function("Text")),
        vector_search_profile_name="myHnswProfile",
    ),
    SearchableField(
        name="metadata",
        type=SearchFieldDataType.String,
        searchable=True,
    ),
    # 用于存储标题的附加字段
    SearchableField(
        name="title",
        type=SearchFieldDataType.String,
        searchable=True,
    ),
    # 用于根据文档来源进行过滤的附加字段
    SimpleField(
        name="source",
        type=SearchFieldDataType.String,
        filterable=True,
    ),
    # 用于存储最后一次文档更新的附加数据字段
    SimpleField(
        name="last_update",
        type=SearchFieldDataType.DateTimeOffset,
        searchable=True,
        filterable=True,
    ),
]
# 添加一个包含新鲜度函数的自定义评分配置文件
sc_name = "scoring_profile"
sc = ScoringProfile(
    name=sc_name,
    functions=[
        FreshnessScoringFunction(
            field_name="last_update",
            parameters=FreshnessScoringParameters(
                boosting_range_start="2022-01-01T00:00:00Z",
                boosting_range_end="2022-12-31T23:59:59Z",
                boosting_function="linear",
                boosting_distance=1,
            ),
        )
    ],
)
```
```python
text_weights=TextWeights(weights={"title": 5}),
function_aggregation="sum",
functions=[
    FreshnessScoringFunction(
        field_name="last_update",
        boost=100,
        parameters=FreshnessScoringParameters(boosting_duration="P2D"),
        interpolation="linear",
    )
],
)
index_name = "langchain-vector-demo-custom-scoring-profile"
vector_store: AzureSearch = AzureSearch(
    azure_search_endpoint=vector_store_address,
    azure_search_key=vector_store_password,
    index_name=index_name,
    embedding_function=embeddings.embed_query,
    fields=fields,
    scoring_profiles=[sc],
    default_scoring_profile=sc_name,
)
```
```python
# 添加具有不同 last_update 的相同数据以展示 Scoring Profile 的效果
from datetime import datetime, timedelta
today = datetime.utcnow().strftime("%Y-%m-%dT%H:%M:%S-00:00")
yesterday = (datetime.utcnow() - timedelta(days=1)).strftime("%Y-%m-%dT%H:%M:%S-00:00")
one_month_ago = (datetime.utcnow() - timedelta(days=30)).strftime(
    "%Y-%m-%dT%H:%M:%S-00:00"
)
vector_store.add_texts(
    ["测试 1", "测试 1", "测试 1"],
    [
        {
            "title": "标题 1",
            "source": "来源1",
            "random": "10290",
            "last_update": today,
        },
        {
            "title": "标题 1",
            "source": "来源1",
            "random": "48392",
            "last_update": yesterday,
        },
        {
            "title": "标题 1",
            "source": "来源1",
            "random": "32893",
            "last_update": one_month_ago,
        },
    ],
)
```
```output

['NjUwNGQ5ZDUtMGVmMy00OGM4LWIxMGYtY2Y2MDFmMTQ0MjE5',

 'NWFjN2YwY2UtOWQ4Yi00OTNhLTg2MGEtOWE0NGViZTVjOGRh',

 'N2Y2NWUyZjctMDBjZC00OGY4LWJlZDEtNTcxYjQ1MmI1NjYx']

```
```python
res = vector_store.similarity_search(query="测试 1", k=3, search_type="similarity")
res
```
```output

[Document(page_content='测试 1', metadata={'title': '标题 1', 'source': '来源1', 'random': '32893', 'last_update': '2024-01-24T22:18:51-00:00'}),

 Document(page_content='测试 1', metadata={'title': '标题 1', 'source': '来源1', 'random': '48392', 'last_update': '2024-02-22T22:18:51-00:00'}),

 Document(page_content='测试 1', metadata={'title': '标题 1', 'source': '来源1', 'random': '10290', 'last_update': '2024-02-23T22:18:51-00:00'})]

```