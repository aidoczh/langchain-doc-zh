# 河马

[Transwarp Hippo](https://www.transwarp.cn/en/subproduct/hippo) 是一种企业级的云原生分布式向量数据库，支持存储、检索和管理海量基于向量的数据集。它高效地解决了向量相似性搜索和高密度向量聚类等问题。`Hippo` 具有高可用性、高性能和易扩展性的特点。它具有多个向量搜索索引、数据分区和分片、数据持久化、增量数据摄取、向量标量字段过滤和混合查询等多种功能。它能够有效满足企业对海量向量数据的高实时搜索需求。

## 入门

这里唯一的先决条件是从 OpenAI 网站获取一个 API 密钥。确保您已经启动了一个 Hippo 实例。

## 安装依赖

首先，我们需要安装一些依赖项，例如 OpenAI、Langchain 和 Hippo-API。请注意，您应该安装适合您环境的相应版本。

```python
%pip install --upgrade --quiet  langchain tiktoken langchain-openai
%pip install --upgrade --quiet  hippo-api==1.1.0.rc3
```

```output
Requirement already satisfied: hippo-api==1.1.0.rc3 in /Users/daochengzhang/miniforge3/envs/py310/lib/python3.10/site-packages (1.1.0rc3)
Requirement already satisfied: pyyaml>=6.0 in /Users/daochengzhang/miniforge3/envs/py310/lib/python3.10/site-packages (from hippo-api==1.1.0.rc3) (6.0.1)
```

注意：Python 版本需要 >=3.8。

## 最佳实践

### 导入依赖包

```python
import os
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.hippo import Hippo
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

### 加载知识文档

```python
os.environ["OPENAI_API_KEY"] = "YOUR OPENAI KEY"
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
```

### 分割知识文档

这里我们使用 Langchain 的 CharacterTextSplitter 进行分割。分隔符是句号。分割后的文本段不超过 1000 个字符，重复字符数为 0。

```python
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

### 声明嵌入模型

下面，我们使用 Langchain 的 OpenAIEmbeddings 方法创建 OpenAI 或 Azure 嵌入模型。

```python
# openai
embeddings = OpenAIEmbeddings()
# azure
# embeddings = OpenAIEmbeddings(
#     openai_api_type="azure",
#     openai_api_base="x x x",
#     openai_api_version="x x x",
#     model="x x x",
#     deployment="x x x",
#     openai_api_key="x x x"
# )
```

### 声明 Hippo 客户端

```python
HIPPO_CONNECTION = {"host": "IP", "port": "PORT"}
```

### 存储文档

```python
print("input...")
# insert docs
vector_store = Hippo.from_documents(
    docs,
    embedding=embeddings,
    table_name="langchain_test",
    connection_args=HIPPO_CONNECTION,
)
print("success")
```

```output
input...
success
```

### 进行基于知识的问答

#### 创建大型语言问答模型

下面，我们分别使用 AzureChatOpenAI 和 ChatOpenAI 方法从 Langchain 创建 OpenAI 或 Azure 大型语言问答模型。

```python
# llm = AzureChatOpenAI(
#     openai_api_base="x x x",
#     openai_api_version="xxx",
#     deployment_name="xxx",
#     openai_api_key="xxx",
#     openai_api_type="azure"
# )
llm = ChatOpenAI(openai_api_key="YOUR OPENAI KEY", model_name="gpt-3.5-turbo-16k")
```

### 根据问题获取相关知识

```python
query = "Please introduce COVID-19"
# query = "Please introduce Hippo Core Architecture"
# query = "What operations does the Hippo Vector Database support for vector data?"
# query = "Does Hippo use hardware acceleration technology? Briefly introduce hardware acceleration technology."
# 从知识库中检索相似内容，获取前两个最相似的文本。
res = vector_store.similarity_search(query, 2)
content_list = [item.page_content for item in res]
text = "".join(content_list)
```

### 构建提示模板

```python
prompt = f"""
Please use the content of the following [Article] to answer my question. If you don't know, please say you don't know, and the answer should be concise."
[Article]:{text}
Please answer this question in conjunction with the above article:{query}
"""
```

### 等待大型语言模型生成答案

```python
response_with_hippo = llm.predict(prompt)
print(f"response_with_hippo:{response_with_hippo}")
response = llm.predict(query)
print("==========================================")
print(f"response_without_hippo:{response}")
```

```output
response_with_hippo:COVID-19 是一种影响我们生活方方面面超过两年的病毒。它具有极强的传染性，并且容易发生变异，要求我们保持警惕以遏制其传播。然而，由于取得的进展和个体的韧性，我们现在能够安全地前行，恢复更正常的生活节奏。
==========================================
response_without_hippo:COVID-19 是一种由新型冠状病毒SARS-CoV-2引起的传染性呼吸道疾病。它首次于2019年12月在中国武汉被发现，此后迅速在全球传播，导致了一场大流行。该病毒主要通过呼吸道飞沫传播，当感染者咳嗽、打喷嚏、说话或呼吸时会释放呼吸道飞沫，也可以通过接触被污染的表面然后触摸面部而传播。COVID-19 的症状包括发热、咳嗽、呼吸急促、疲劳、肌肉或身体疼痛、咽喉痛、味觉或嗅觉丧失、头痛，以及在严重病例中可能出现肺炎和器官衰竭。虽然大多数人经历轻度至中度症状，但它可能导致严重疾病甚至死亡，尤其是在老年人和患有基础健康问题的人群中。为了遏制病毒的传播，全球范围内已实施了各种预防措施，包括保持社交距离、佩戴口罩、保持良好的手部卫生以及进行疫苗接种工作。
```