#  Activeloop Deep Lake

[Activeloop Deep Lake](https://docs.activeloop.ai/) 是一个多模态向量存储库，用于存储嵌入向量及其元数据，包括文本、Jsons、图像、音频、视频等。它可以将数据保存在本地、云端或Activeloop存储中。它执行包括嵌入向量及其属性在内的混合搜索。

这个笔记本展示了与 `Activeloop Deep Lake` 相关的基本功能。虽然 `Deep Lake` 可以存储嵌入向量，但它也能够存储任何类型的数据。它是一个具有版本控制、查询引擎和流式数据加载器的无服务器数据湖，可供深度学习框架使用。

欲了解更多信息，请参阅 Deep Lake [文档](https://docs.activeloop.ai) 或 [API 参考](https://docs.deeplake.ai)。

## 设置

```python
%pip install --upgrade --quiet langchain-openai 'deeplake[enterprise]' tiktoken
```

## Activeloop 提供的示例

[与 LangChain 集成](https://docs.activeloop.ai/tutorials/vector-store/deep-lake-vector-store-in-langchain)。

## 本地 Deep Lake

```python
from langchain_community.vectorstores import DeepLake
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
activeloop_token = getpass.getpass("activeloop token:")
embeddings = OpenAIEmbeddings()
```

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

### 创建本地数据集

在 `./deeplake/` 本地创建数据集，然后运行相似性搜索。Deep Lake + LangChain 集成在幕后使用 Deep Lake 数据集，因此 `dataset` 和 `vector store` 可以互换使用。要在您自己的云端或Deep Lake 存储中创建数据集，请[相应调整路径](https://docs.activeloop.ai/storage-and-credentials/storage-options)。

```python
db = DeepLake(dataset_path="./my_deeplake/", embedding=embeddings, overwrite=True)
db.add_documents(docs)
# 或者更简洁地
# db = DeepLake.from_documents(docs, dataset_path="./my_deeplake/", embedding=embeddings, overwrite=True)
```

### 查询数据集

```python
query = "总统关于 Ketanji Brown Jackson 说了什么"
docs = db.similarity_search(query)
```

```output
Dataset(path='./my_deeplake/', tensors=['embedding', 'id', 'metadata', 'text'])
  tensor      htype      shape      dtype  compression
  -------    -------    -------    -------  ------- 
 embedding  embedding  (42, 1536)  float32   None   
    id        text      (42, 1)      str     None   
 metadata     json      (42, 1)      str     None   
   text       text      (42, 1)      str     None
```

要避免每次打印数据集摘要，您可以在初始化 VectorStore 时指定 verbose=False。

```python
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《揭示法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶尔——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶尔法官，感谢您的服务。
总统拥有的最严肃的宪法责任之一是提名某人担任美国最高法院法官。
我在4天前做到了，当时我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续布雷耶尔法官的卓越传统。
```

稍后，您可以重新加载数据集而无需重新计算嵌入向量。

```python
db = DeepLake(dataset_path="./my_deeplake/", embedding=embeddings, read_only=True)
docs = db.similarity_search(query)
```

```output
Deep Lake Dataset in ./my_deeplake/ already exists, loading from the storage
```

目前，Deep Lake 是单写多读的。设置 `read_only=True` 有助于避免获取写入锁。

### 检索问答

```python
from langchain.chains import RetrievalQA
from langchain_openai import OpenAIChat
qa = RetrievalQA.from_chain_type(
    llm=OpenAIChat(model="gpt-3.5-turbo"),
    chain_type="stuff",
    retriever=db.as_retriever(),
)
```

```output
/home/ubuntu/langchain_activeloop/langchain/libs/langchain/langchain/llms/openai.py:786: UserWarning: You are trying to use a chat model. This way of initializing it is no longer supported. Instead, please use: `from langchain_openai import ChatOpenAI`
  warnings.warn(
```

```python
查询 = "总统对Ketanji Brown Jackson有什么看法"
qa.run(查询)
```

```output
'总统表示，Ketanji Brown Jackson曾是一位私人执业的顶尖诉讼律师，也曾是一位联邦公共辩护人。她来自一家公立学校教育工作者和警察的家庭。她是一位建设共识的人，并且自被提名以来得到了广泛的支持。'
```

### 元数据中的基于属性的过滤

让我们创建另一个包含文档创建年份元数据的向量存储。

```python
import random
for d in docs:
    d.metadata["year"] = random.randint(2012, 2014)
db = DeepLake.from_documents(
    docs, embeddings, dataset_path="./my_deeplake/", overwrite=True
)
```

```output
``````output
数据集(path='./my_deeplake/', 张量=['embedding', 'id', 'metadata', 'text'])
  张量      类型      形状     数据类型  压缩方式
  -------    -------    -------   -------  ------- 
 embedding  embedding  (4, 1536)  float32   None   
    id        text      (4, 1)      str     None   
 metadata     json      (4, 1)      str     None   
   text       text      (4, 1)      str     None
``````output
```

```python
db.similarity_search(
    "总统对Ketanji Brown Jackson有什么看法",
    filter={"metadata": {"year": 2013}},
)
```

```output
100%|██████████| 4/4 [00:00<00:00, 2936.16it/s]
```

```output
[Document(page_content='今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。而且在此期间，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。司法部长布雷耶，感谢您的服务。\n\n总统担负的最严肃的宪法责任之一就是提名人担任美国最高法院的法官。\n\n而我在4天前就做到了，当时我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013}),
 Document(page_content='曾是一位私人执业的顶尖诉讼律师。曾是一位联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。她是一位建设共识的人。自她被提名以来，她得到了广泛的支持——从警察兄弟会到由民主党和共和党任命的前法官。而如果我们要推进自由和正义，我们需要保护边境并修复移民制度。\n\n我们可以两者兼顾。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测毒品走私。\n\n我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。\n\n我们正在设立专门的移民法官，以便那些逃离迫害和暴力的家庭能够更快地得到审理。\n\n我们正在确保承诺，并支持南美和中美的伙伴，以接纳更多的难民并保护他们自己的边界。', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013}),
 Document(page_content='今晚，我宣布要打击这些公司对美国企业和消费者的过度收费。\n\n随着华尔街公司接管更多的养老院，这些养老院的质量下降了，成本却上升了。\n\n这种情况在我的任期内将结束。\n\n医疗保险将为养老院设定更高的标准，并确保您所爱的人得到他们应得和期望的照顾。\n\n我们还将通过给予工人公平机会、提供更多的培训和学徒制度、根据他们的技能而不是学位来雇佣他们来削减成本并保持经济强劲。\n\n让我们通过通过《工资公平法案》和带薪休假来提高最低工资至每小时15美元，并延长儿童税收抵免，这样就没有人需要在贫困中抚养家庭。\n\n让我们增加Pell Grants并增加我们对历史悠久的黑人大学和研究所的支持，并投资于吉尔——我们的第一夫人，她全职教书——所称之为美国最佳保密的秘密：社区学院。', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013})]
```

### 选择距离函数

距离函数`L2`代表欧几里得距离，`L1`代表核范数，`Max`代表l-无穷距离，`cos`代表余弦相似度，`dot`代表点积

```python
db.similarity_search(
    "总统对Ketanji Brown Jackson有什么看法?", distance_metric="cos"
)
```

```output```

今晚，我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。还有，顺便通过《披露法案》，这样美国人就能知道谁在资助我们的选举。

今晚，我想向一个一生都在为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。

总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。

而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智囊，将继续延续布雷耶司法部长的卓越传统。[20]

![Image](https://example.com/image1.jpg)

（图片链接）

今晚，我呼吁参议院：通过《自由选举法案》（Freedom to Vote Act）。通过《约翰·刘易斯选举权法案》（John Lewis Voting Rights Act）。还有，趁热打铁，通过《揭示法案》（Disclose Act），这样美国人就能知道谁在资助我们的选举。

今晚，我想向一个终身致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶（Stephen Breyer）— 一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。

总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。

4天前，我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊（Ketanji Brown Jackson）。她是我们国家顶尖的法律专家之一，将继续延续布雷耶司法部长的卓越传统。[20]

（图片链接：../../how_to/state_of_the_union.txt）

今晚，我宣布要打击这些对美国企业和消费者进行过度收费的公司。

随着华尔街公司接管更多的养老院，这些养老院的质量下降，成本上升。在我的任期内，这将结束。

医疗保险将为养老院设定更高的标准，确保您的亲人得到他们应得和期望的照顾。

我们还将通过给予工人公平机会、提供更多培训和学徒制度、根据他们的技能而非学位来雇佣他们，来削减成本，保持经济强劲发展。

让我们通过《工资公平法案》（Paycheck Fairness Act）和带薪休假。

将最低工资提高到每小时15美元，并延长儿童税收抵免，这样就不会有人在贫困中抚养家庭。

让我们增加贝尔助学金和增加我们对历史悠久的黑人大学和大学的支持，投资吉尔所说的美国最佳保密：社区学院。[20]

（图片链接：../../how_to/state_of_the_union.txt）

她曾是一位私人执业的顶级诉讼律师，也是一位前联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。她是一位建设共识的人。自从她被提名以来，她得到了广泛的支持 —— 从警察兄弟会到由民主党和共和党任命的前法官。

如果我们要推进自由和正义，我们需要保护边境并修复移民制度。

我们可以两者兼顾。在我们的边境，我们安装了新技术，比如尖端扫描仪，以更好地检测走私毒品。

我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。

我们正在设立专门的移民法官，以便那些逃离迫害和暴力的家庭能够更快地得到审理。

我们正在承诺并支持南美和中美的伙伴，以接纳更多的难民并保护他们的边界。[20]

（图片链接：../../how_to/state_of_the_union.txt）

对于我们的 LGBTQ+ 美国人，让我们最终将两党支持的《平等法案》（Equality Act）送到我的办公桌上。针对跨性别美国人及其家人的一系列州法律的袭击是错误的。

正如我去年所说，特别是对我们年轻的跨性别美国人，作为你们的总统，我将永远支持你们，这样你们就可以做自己，并发挥上帝赋予你们的潜力。

虽然我们经常看起来似乎永远无法达成一致，但事实并非如此。去年，我签署了80项两党法案。从防止政府关门到保护亚裔免受仍然普遍存在的仇恨犯罪，再到改革军事司法。

很快，我们将加强我三十年前首次起草的《反对妇女暴力法》。对我们来说，向国家展示我们可以团结一致，做出重大成就是很重要的。

因此，今晚我提出了一个国家的团结议程。我们可以一起做的四件大事。

首先，战胜阿片类药物流行。[20]

（图片链接：../../how_to/state_of_the_union.txt）

```python
dataset_path = f"hub://{username}/langchain_testing_python"  # 也可以是 ./local/path（在本地更快），s3://bucket/path/to/dataset，gcs://path/to/dataset 等等。
docs = text_splitter.split_documents(documents)
embedding = OpenAIEmbeddings()
db = DeepLake(dataset_path=dataset_path, embedding=embeddings, overwrite=True)
ids = db.add_documents(docs)
```

```output
您的 Deep Lake 数据集已成功创建！
```

```output
Dataset(path='hub://adilkhan/langchain_testing_python', tensors=['embedding', 'id', 'metadata', 'text'])
  tensor      htype      shape      dtype  compression
  -------    -------    -------    -------  ------- 
 embedding  embedding  (42, 1536)  float32   None   
    id        text      (42, 1)      str     None   
 metadata     json      (42, 1)      str     None   
   text       text      (42, 1)      str     None
```

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
print(docs[0].page_content)
```

```output
今晚，我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法案》，这样美国人就能知道谁在资助我们的选举。
今晚，我想向一个致力于为这个国家服务的人表示敬意：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者和即将退休的美国最高法院法官。布雷耶法官，感谢您的服务。
总统最重要的宪法责任之一是提名人选担任美国最高法院法官。
我在4天前做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律专家，将继续延续布雷耶法官的卓越传统。
```

#### `tensor_db` 执行选项

为了使用 Deep Lake 的托管张量数据库，需要在创建向量存储时指定运行时参数为 {'tensor_db': True}。这个配置使得查询在托管张量数据库上执行，而不是在客户端上执行。需要注意的是，这个功能对于本地存储或内存中的数据集不适用。如果已经在托管张量数据库之外创建了向量存储，可以按照规定的步骤将其转移到托管张量数据库中。

```python
# 嵌入并存储文本
username = "<USERNAME_OR_ORG>"  # 在 app.activeloop.ai 上的用户名
dataset_path = f"hub://{username}/langchain_testing"
docs = text_splitter.split_documents(documents)
embedding = OpenAIEmbeddings()
db = DeepLake(
    dataset_path=dataset_path,
    embedding=embeddings,
    overwrite=True,
    runtime={"tensor_db": True},
)
ids = db.add_documents(docs)
```

```output
您的 Deep Lake 数据集已成功创建！
```

```output
Dataset(path='hub://adilkhan/langchain_testing', tensors=['embedding', 'id', 'metadata', 'text'])
  tensor      htype      shape      dtype  compression
  -------    -------    -------    -------  ------- 
 embedding  embedding  (42, 1536)  float32   None   
    id        text      (42, 1)      str     None   
 metadata     json      (42, 1)      str     None   
   text       text      (42, 1)      str     None
```

### TQL 搜索

此外，还支持使用 Deep Lake 的张量查询语言（TQL）在 `similarity_search` 方法中指定查询。

```python
search_id = db.vectorstore.dataset.id[0].numpy()
```

```python
search_id[0]
```

```output
'8a6ff326-3a85-11ee-b840-13905694aaaf'
```

```python
docs = db.similarity_search(
    query=None,
    tql=f"SELECT * WHERE id == '{search_id[0]}'",
)
```

```python
db.vectorstore.summary()
```

```output
Dataset(path='hub://adilkhan/langchain_testing', tensors=['embedding', 'id', 'metadata', 'text'])
  tensor      htype      shape      dtype  compression
  -------    -------    -------    -------  ------- 
 embedding  embedding  (42, 1536)  float32   None   
    id        text      (42, 1)      str     None   
 metadata     json      (42, 1)      str     None   
   text       text      (42, 1)      str     None
```

### 在 AWS S3 上创建向量存储

```python
dataset_path = "s3://BUCKET/langchain_test"  # 也可以是 ./local/path（在本地更快），hub://bucket/path/to/dataset，gcs://path/to/dataset 等等。
embedding = OpenAIEmbeddings()
db = DeepLake.from_documents(
    docs,
    dataset_path=dataset_path,
    embedding=embeddings,
    overwrite=True,
    creds={
        "aws_access_key_id": os.environ["AWS_ACCESS_KEY_ID"],
        "aws_secret_access_key": os.environ["AWS_SECRET_ACCESS_KEY"],
```

```python
"aws_session_token": os.environ["AWS_SESSION_TOKEN"],  # 可选
},
)
```

```output
s3://hub-2.0-datasets-n/langchain_test 成功加载。
``````output
评估摄入：100%|██████████| 1/1 [00:10<00:00
\
``````output
数据集(path='s3://hub-2.0-datasets-n/langchain_test', 张量=['embedding', 'ids', 'metadata', 'text'])
  张量       类型       形状       数据类型  压缩方式
  -------   -------   -------   -------  ------- 
 embedding  通用      (4, 1536)  float32   无   
    ids      文本      (4, 1)      字符串     无   
 metadata    json     (4, 1)      字符串     无   
   text      文本      (4, 1)      字符串     无
``````output
```

## Deep Lake API

您可以在 `db.vectorstore` 中访问 Deep Lake 数据集

```python
# 获取数据集的结构
db.vectorstore.summary()
```

```output
数据集(path='hub://adilkhan/langchain_testing', 张量=['embedding', 'id', 'metadata', 'text'])
  张量       类型       形状       数据类型  压缩方式
  -------   -------   -------   -------  ------- 
 embedding  嵌入      (42, 1536)  float32   无   
    id       文本      (42, 1)      字符串     无   
 metadata    json      (42, 1)      字符串     无   
   text       文本      (42, 1)      字符串     无
```

```python
# 获取嵌入的 numpy 数组
embeds = db.vectorstore.dataset.embedding.numpy()
```

### 将本地数据集转移到云端

将已创建的数据集复制到云端。您也可以从云端转移到本地。

```python
import deeplake
username = "davitbun"  # 您在 app.activeloop.ai 上的用户名
source = f"hub://{username}/langchain_testing"  # 可以是本地、s3、gcs 等
destination = f"hub://{username}/langchain_test_copy"  # 可以是本地、s3、gcs 等
deeplake.deepcopy(src=source, dest=destination, overwrite=True)
```

```output
复制数据集：100%|██████████| 56/56 [00:38<00:00
``````output
此数据集可以通过 ds.visualize() 在 Jupyter Notebook 中或在 https://app.activeloop.ai/davitbun/langchain_test_copy 中进行可视化
您的 Deep Lake 数据集已成功创建！
该数据集是私有的，因此请确保您已登录！
```

```output
数据集(path='hub://davitbun/langchain_test_copy', 张量=['embedding', 'ids', 'metadata', 'text'])
```

```python
db = DeepLake(dataset_path=destination, embedding=embeddings)
db.add_documents(docs)
```

```output
``````output
此数据集可以通过 ds.visualize() 在 Jupyter Notebook 中或在 https://app.activeloop.ai/davitbun/langchain_test_copy 中进行可视化
``````output
/
``````output
hub://davitbun/langchain_test_copy 成功加载。
``````output
Deep Lake 数据集在 hub://davitbun/langchain_test_copy 已存在，从存储加载
``````output
数据集(path='hub://davitbun/langchain_test_copy', 张量=['embedding', 'ids', 'metadata', 'text'])
  张量       类型       形状       数据类型  压缩方式
  -------   -------   -------   -------  ------- 
 embedding  通用      (4, 1536)  float32   无   
    ids      文本      (4, 1)      字符串     无   
 metadata    json     (4, 1)      字符串     无   
   text      文本      (4, 1)      字符串     无
``````output
评估摄入：100%|██████████| 1/1 [00:31<00:00
-
``````output
数据集(path='hub://davitbun/langchain_test_copy', 张量=['embedding', 'ids', 'metadata', 'text'])
  张量       类型       形状       数据类型  压缩方式
  -------   -------   -------   -------  ------- 
 embedding  通用      (8, 1536)  float32   无   
    ids      文本      (8, 1)      字符串     无   
 metadata    json     (8, 1)      字符串     无   
   text      文本      (8, 1)      字符串     无
``````output
```

```output
['ad42f3fe-e188-11ed-b66d-41c5f7b85421',
 'ad42f3ff-e188-11ed-b66d-41c5f7b85421',
 'ad42f400-e188-11ed-b66d-41c5f7b85421',
 'ad42f401-e188-11ed-b66d-41c5f7b85421']
```