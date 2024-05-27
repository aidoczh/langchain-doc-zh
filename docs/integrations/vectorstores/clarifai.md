# Clarifai

[Clarifai](https://www.clarifai.com/) 是一个提供完整 AI 生命周期的 AI 平台，包括数据探索、数据标注、模型训练、评估和推理。上传输入后，Clarifai 应用程序可以用作向量数据库。

本文介绍了如何使用与 `Clarifai` 向量数据库相关的功能。示例演示了文本语义搜索的能力。Clarifai 还支持图像、视频帧和本地化搜索（参见 [Rank](https://docs.clarifai.com/api-guide/search/rank)）以及属性搜索（参见 [Filter](https://docs.clarifai.com/api-guide/search/filter)）。

要使用 Clarifai，您必须拥有一个帐户和个人访问令牌（PAT）密钥。

[在此处](https://clarifai.com/settings/security) 获取或创建 PAT。

# 依赖项

```python
# 安装所需依赖项
%pip install --upgrade --quiet clarifai
```

# 导入

在这里，我们将设置个人访问令牌。您可以在平台的设置/安全性下找到您的 PAT。

```python
# 请登录并从 https://clarifai.com/settings/security 获取您的 API 密钥
from getpass import getpass
CLARIFAI_PAT = getpass()
```

```output
 ········
```

```python
# 导入所需模块
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Clarifai
from langchain_text_splitters import CharacterTextSplitter
```

# 设置

设置用户 ID 和应用程序 ID，其中将上传文本数据。注意：在创建该应用程序时，请选择适当的基础工作流程来索引您的文本文档，例如语言理解工作流程。

您首先需要在 [Clarifai](https://clarifai.com/login) 上创建一个帐户，然后创建一个应用程序。

```python
USER_ID = "USERNAME_ID"
APP_ID = "APPLICATION_ID"
NUMBER_OF_DOCS = 2
```

## 来自文本

从文本列表创建 Clarifai 向量数据库。此部分将每个文本及其相应的元数据上传到 Clarifai 应用程序。然后可以使用 Clarifai 应用程序进行语义搜索以找到相关的文本。

```python
texts = [
    "我真的很喜欢和你在一起",
    "我讨厌和我的狗在一起",
    "我想去跑步",
    "昨天我去看电影了",
    "我喜欢和朋友们一起踢足球",
]
metadatas = [
    {"id": i, "text": text, "source": "book 1", "category": ["books", "modern"]}
    for i, text in enumerate(texts)
]
```

或者您可以选择为输入提供自定义输入 ID。

```python
idlist = ["text1", "text2", "text3", "text4", "text5"]
metadatas = [
    {"id": idlist[i], "text": text, "source": "book 1", "category": ["books", "modern"]}
    for i, text in enumerate(texts)
]
```

```python
# 可以使用 PAT 初始化 Clarifai 向量存储!
clarifai_vector_db = Clarifai(
    user_id=USER_ID,
    app_id=APP_ID,
    number_of_docs=NUMBER_OF_DOCS,
)
```

将数据上传到 Clarifai 应用程序。

```python
# 带有元数据和自定义输入 ID 的上传。
response = clarifai_vector_db.add_texts(texts=texts, ids=idlist, metadatas=metadatas)
# 上传没有元数据（不推荐）- 因为您将无法根据元数据执行搜索操作。
# 自定义输入 ID（可选）
response = clarifai_vector_db.add_texts(texts=texts)
```

您可以创建一个 Clarifai 向量数据库存储，并将所有输入直接导入到您的应用程序中。

```python
clarifai_vector_db = Clarifai.from_texts(
    user_id=USER_ID,
    app_id=APP_ID,
    texts=texts,
    metadatas=metadatas,
)
```

使用相似性搜索函数搜索相似的文本。

```python
docs = clarifai_vector_db.similarity_search("我想见到你")
docs
```

```output
[Document(page_content='我真的很喜欢和你在一起', metadata={'text': '我真的很喜欢和你在一起', 'id': 'text1', 'source': 'book 1', 'category': ['books', 'modern']})]
```

您还可以通过元数据筛选搜索结果。

```python
# 在应用程序中可以进行很多强大的过滤操作，利用元数据过滤器。
# 此示例将将相似查询限制为仅具有键为 "source" 的值与 "book 1" 匹配的文本。
book1_similar_docs = clarifai_vector_db.similarity_search(
    "我想见到你", filter={"source": "book 1"}
)
# 您还可以在输入的元数据中使用列表，然后选择与列表中的项匹配的内容。这对于下面的类别很有用：
book_category_similar_docs = clarifai_vector_db.similarity_search(
    "我想见到你", filter={"category": ["books"]}
)
```

## 来自文档

创建一个 Clarifai 向量存储库，其中包含一系列文档。这一部分将每个文档及其相应的元数据上传到 Clarifai 应用程序。然后可以使用 Clarifai 应用程序进行语义搜索，以找到相关文档。

```python
loader = TextLoader("your_local_file_path.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

```python
USER_ID = "USERNAME_ID"
APP_ID = "APPLICATION_ID"
NUMBER_OF_DOCS = 4
```

创建一个 Clarifai 向量数据库类，并将所有文档导入 Clarifai 应用程序。

```python
clarifai_vector_db = Clarifai.from_documents(
    user_id=USER_ID,
    app_id=APP_ID,
    documents=docs,
    number_of_docs=NUMBER_OF_DOCS,
)
```

```python
docs = clarifai_vector_db.similarity_search("与人口相关的文本")
docs
```

## 从现有应用程序中

在 Clarifai 中，我们有很好的工具，可以通过 API 或 UI 向应用程序（实质上是项目）添加数据。大多数用户在与 LangChain 互动之前通常已经这样做了，因此此示例将使用现有应用程序中的数据来执行搜索。请查看我们的 [API 文档](https://docs.clarifai.com/api-guide/data/create-get-update-delete) 和 [UI 文档](https://docs.clarifai.com/portal-guide/data)。然后可以使用 Clarifai 应用程序进行语义搜索，以找到相关文档。

```python
USER_ID = "USERNAME_ID"
APP_ID = "APPLICATION_ID"
NUMBER_OF_DOCS = 4
```

```python
clarifai_vector_db = Clarifai(
    user_id=USER_ID,
    app_id=APP_ID,
    number_of_docs=NUMBER_OF_DOCS,
)
```

```python
docs = clarifai_vector_db.similarity_search(
    "与弹药和威尔逊总统相关的文本"
)
```

```python
docs[0].page_content
```

```output
"总体上被誉为世界民主领袖的威尔逊总统，在战后的大和平会议上，为文明阐述了反对专制主义的论点。总统领导美国代表团参加了那次世界重建的大会。与他一同出席会议的代表有国务卿罗伯特·兰辛、前驻法国和意大利大使亨利·怀特、爱德华·豪斯和塔斯克·H·布利斯将军。\n在与和平会议同时举行的巴黎国际劳工大会上，代表美国劳工的有美国劳联主席塞缪尔·高普斯、美国煤矿工会秘书长威廉·格林、管道工会主席约翰·阿尔派恩、国际花岗岩切割工会主席詹姆斯·邓肯、联合木工和结构工人会主席弗兰克·达菲以及美国劳联秘书弗兰克·莫里森。\n估算每个协约国在伟大胜利中的份额，人类将得出结论，按照战前人口和财富的比例，首先感受到战争冲击的国家，比利时、塞尔维亚、波兰和法国承担了最沉重的代价。这四个国家都是庞大军队的战场，这些军队在曾经肥沃的土地和曾经繁荣的城镇上进行着血腥的争夺。\n比利时，人口800万，伤亡人数超过35万；法国，其伤亡人数达到400万，占人口（包括殖民地）的90,000,000，实际上是世界上的殉道国。她的勇敢士兵向世界展示了人们如何在保卫家园和自由时愉快地牺牲。庞大的俄罗斯，包括不幸的波兰，其伤亡人数达到700万，占全国总人口的180,000,000。美国在战争的十九个月中，伤亡人数为236,117；其中有53,169人阵亡或死于疾病；179,625人受伤；3,323人被俘或失踪。"
```