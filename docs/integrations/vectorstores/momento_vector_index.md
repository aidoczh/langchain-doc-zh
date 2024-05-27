# Momento 矢量索引（MVI）

[MVI](https://gomomento.com)：最高效、最易用的无服务器矢量索引，适用于您的数据。要开始使用 MVI，只需注册一个帐户。无需处理基础架构、管理服务器或担心扩展性。MVI 是一个可以自动扩展以满足您需求的服务。

要注册并访问 MVI，请访问[Momento 控制台](https://console.gomomento.com)。

# 设置

## 安装先决条件

您将需要：

- [`momento`](https://pypi.org/project/momento/) 包用于与 MVI 进行交互，以及

- 与 OpenAI API 进行交互的 openai 包。

- 用于对文本进行标记的 tiktoken 包。

```python
%pip install --upgrade --quiet  momento langchain-openai tiktoken
```

## 输入 API 密钥

```python
import getpass
import os
```

### Momento：用于索引数据

访问[Momento 控制台](https://console.gomomento.com)获取您的 API 密钥。

```python
os.environ["MOMENTO_API_KEY"] = getpass.getpass("Momento API Key: ")
```

### OpenAI：用于文本嵌入

```python
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
```

# 加载您的数据

这里我们使用 Langchain 的示例数据集，即国情咨文。

首先，我们加载相关模块：

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import MomentoVectorIndex
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

然后，我们加载数据：

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
len(documents)
```

```output
1
```

请注意，数据是一个大文件，因此只有一个文档：

```python
len(documents[0].page_content)
```

```output
38539
```

因为这是一个大文本文件，我们将其分割成块以进行问题回答。这样，用户的问题将从最相关的块中得到回答：

```python
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
len(docs)
```

```output
42
```

# 索引您的数据

索引您的数据就像实例化 `MomentoVectorIndex` 对象一样简单。这里我们使用 `from_documents` 助手来实例化和索引数据：

```python
vector_db = MomentoVectorIndex.from_documents(
    docs, OpenAIEmbeddings(), index_name="sotu"
)
```

这将使用您的 API 密钥连接到 Momento 矢量索引服务并索引数据。如果索引之前不存在，则此过程将为您创建它。数据现在可以进行搜索。

# 查询您的数据

## 直接针对索引提出问题

查询数据的最直接方式是针对索引进行搜索。我们可以使用 `VectorStore` API 来进行如下操作：

```python
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = vector_db.similarity_search(query)
```

```python
docs[0].page_content
```

```output
'今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯投票权法案》。而且，趁机通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个致力于为这个国家服务的人表示敬意：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院的退休法官。布雷耶法官，感谢您的服务。\n\n总统的最严肃的宪法责任之一是提名某人担任美国最高法院的法官。\n\n而我在 4 天前就做到了，当时我提名了联邦上诉法院法官 Ketanji Brown Jackson。她是我们国家最顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。'
```

虽然这包含了有关 Ketanji Brown Jackson 的相关信息，但我们没有简洁易懂的答案。我们将在下一节中解决这个问题。

## 使用 LLM 生成流畅的答案

在 MVI 中索引了数据后，我们可以与任何利用矢量相似性搜索的链进行集成。这里我们使用 `RetrievalQA` 链来演示如何从索引的数据中回答问题。

首先，我们加载相关模块：

```python
from langchain.chains import RetrievalQA
from langchain_openai import ChatOpenAI
```

然后，我们实例化检索 QA 链：

```python
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
qa_chain = RetrievalQA.from_chain_type(llm, retriever=vector_db.as_retriever())
```

```python
qa_chain({"query": "总统对 Ketanji Brown Jackson 说了什么?"})
```

```output
{'query': '总统对 Ketanji Brown Jackson 说了什么?',
 'result': '总统表示，他提名了联邦上诉法院法官 Ketanji Brown Jackson 担任美国最高法院的法官。他形容她是全国最顶尖的法律智慧之一，并提到她得到了来自各方的广泛支持，包括警察协会和由民主党和共和党任命的前法官。'}
```

# 下一步

到此为止！您已经对数据进行了索引，并可以使用 Momento 矢量索引进行查询。您可以使用相同的索引从支持矢量相似性搜索的任何链中查询您的数据。

使用 Momento，您不仅可以对矢量数据进行索引，还可以缓存您的 API 调用并存储您的聊天消息历史。查看其他 Momento langchain 集成以了解更多信息。

要了解有关 Momento 矢量索引的更多信息，请访问[Momento 文档](https://docs.gomomento.com)。