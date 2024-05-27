# Yellowbrick

[Yellowbrick](https://yellowbrick.com/yellowbrick-data-warehouse/) 是一种弹性的、大规模并行处理（MPP）的 SQL 数据库，可在云端和本地运行，使用 Kubernetes 进行规模、弹性和云端可移植性。Yellowbrick 旨在解决最大和最复杂的业务关键数据仓库使用案例。Yellowbrick 提供的规模效率还使其能够作为高性能和可扩展的向量数据库，用于存储和搜索带有 SQL 的向量。

## 使用 Yellowbrick 作为 ChatGpt 的向量存储

本教程演示了如何创建一个简单的聊天机器人，该机器人由 ChatGpt 支持，并使用 Yellowbrick 作为向量存储以支持检索增强生成（RAG）。您需要以下内容：

1. [Yellowbrick 沙盒](https://cloudlabs.yellowbrick.com/) 上的账户

2. 来自 [OpenAI](https://platform.openai.com/) 的 API 密钥

教程分为五个部分。首先，我们将使用 langchain 创建一个基准聊天机器人，以与 ChatGpt 进行交互，而不需要向量存储。其次，我们将在 Yellowbrick 中创建一个嵌入表，该表将表示向量存储。第三，我们将加载一系列文档（Yellowbrick 手册的管理章节）。第四，我们将创建这些文档的向量表示，并存储在 Yellowbrick 表中。最后，我们将向改进后的聊天机器人发送相同的查询，以查看结果。

```python
# 安装所有所需的库
%pip install --upgrade --quiet  langchain
%pip install --upgrade --quiet  langchain-openai
%pip install --upgrade --quiet  psycopg2-binary
%pip install --upgrade --quiet  tiktoken
```

## 设置：输入用于连接到 Yellowbrick 和 OpenAI API 的信息

我们的聊天机器人通过 langchain 库与 ChatGpt 集成，因此您首先需要来自 OpenAI 的 API 密钥：

要获取 OpenAI 的 API 密钥：

1. 在 https://platform.openai.com/ 注册

2. 添加支付方式 - 您不太可能超出免费配额

3. 创建 API 密钥

您还需要在注册 Yellowbrick 沙盒账户时，从欢迎邮件中获取您的用户名、密码和数据库名称。

以下内容应修改为包含您的 Yellowbrick 数据库和 OpenAPI 密钥的信息：

```python
# 修改这些值以匹配您的 Yellowbrick 沙盒和 OpenAI API 密钥
YBUSER = "[SANDBOX USER]"
YBPASSWORD = "[SANDBOX PASSWORD]"
YBDATABASE = "[SANDBOX_DATABASE]"
YBHOST = "trialsandbox.sandbox.aws.yellowbrickcloud.com"
OPENAI_API_KEY = "[OPENAI API KEY]"
```

```python
# 导入库并设置密钥/登录信息
import os
import pathlib
import re
import sys
import urllib.parse as urlparse
from getpass import getpass
import psycopg2
from IPython.display import Markdown, display
from langchain.chains import LLMChain, RetrievalQAWithSourcesChain
from langchain_community.vectorstores import Yellowbrick
from langchain_core.documents import Document
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
# 建立到 Yellowbrick 的连接参数。如果您已经注册了沙盒，请在此处填写欢迎邮件中的信息：
yellowbrick_connection_string = (
    f"postgres://{urlparse.quote(YBUSER)}:{YBPASSWORD}@{YBHOST}:5432/{YBDATABASE}"
)
YB_DOC_DATABASE = "sample_data"
YB_DOC_TABLE = "yellowbrick_documentation"
embedding_table = "my_embeddings"
# OpenAI 的 API 密钥。在 https://platform.openai.com 注册
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
from langchain_core.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    SystemMessagePromptTemplate,
)
```

## 第 1 部分：创建一个基准聊天机器人，由 ChatGpt 支持，没有向量存储

我们将使用 langchain 查询 ChatGPT。由于没有向量存储，ChatGPT 将没有上下文来回答问题。

```python
# 设置聊天模型和特定提示
system_template = """If you don't know the answer, Make up your best guess."""
messages = [
    SystemMessagePromptTemplate.from_template(system_template),
    HumanMessagePromptTemplate.from_template("{question}"),
]
prompt = ChatPromptTemplate.from_messages(messages)
chain_type_kwargs = {"prompt": prompt}
llm = ChatOpenAI(
    model_name="gpt-3.5-turbo",  # 如果您有访问权限，修改 model_name 为 GPT-4
    temperature=0,
    max_tokens=256,
)
chain = LLMChain(
    llm=llm,
    prompt=prompt,
    verbose=False,
)
def print_result_simple(query):
    result = chain(query)
    output_text = f"""### Question:
  {query}
  ### Answer: 
  {result['text']}
    """
    display(Markdown(output_text))
# 使用链式查询
print_result_simple("How many databases can be in a Yellowbrick Instance?")
print_result_simple("What's an easy way to add users in bulk to Yellowbrick?")
```

## 第二部分：连接到 Yellowbrick 并创建嵌入表

要将文档嵌入加载到 Yellowbrick 中，您应该创建自己的表来存储它们。请注意，该表所在的 Yellowbrick 数据库必须采用 UTF-8 编码。

在 UTF-8 数据库中创建一个表，使用以下模式，并提供您选择的表名：

```python
# 建立到 Yellowbrick 数据库的连接
try:
    conn = psycopg2.connect(yellowbrick_connection_string)
except psycopg2.Error as e:
    print(f"连接到数据库时出错：{e}")
    exit(1)
# 使用连接创建一个游标对象
cursor = conn.cursor()
# 定义创建表的 SQL 语句
create_table_query = f"""
CREATE TABLE IF NOT EXISTS {embedding_table} (
    doc_id uuid NOT NULL,
    embedding_id smallint NOT NULL,
    embedding double precision NOT NULL
)
DISTRIBUTE ON (doc_id);
truncate table {embedding_table};
"""
# 执行创建表的 SQL 查询
try:
    cursor.execute(create_table_query)
    print(f"成功创建表 '{embedding_table}'！")
except psycopg2.Error as e:
    print(f"创建表时出错：{e}")
    conn.rollback()
# 提交更改并关闭游标和连接
conn.commit()
cursor.close()
conn.close()
```

## 第三部分：从 Yellowbrick 中提取要索引的文档

从现有的 Yellowbrick 表中提取文档路径和内容。我们将使用这些文档来在下一步中创建嵌入。

```python
yellowbrick_doc_connection_string = (
    f"postgres://{urlparse.quote(YBUSER)}:{YBPASSWORD}@{YBHOST}:5432/{YB_DOC_DATABASE}"
)
print(yellowbrick_doc_connection_string)
# 建立到 Yellowbrick 数据库的连接
conn = psycopg2.connect(yellowbrick_doc_connection_string)
# 创建一个游标对象
cursor = conn.cursor()
# 查询以选择表中的所有文档
query = f"SELECT path, document FROM {YB_DOC_TABLE}"
# 执行查询
cursor.execute(query)
# 获取所有文档
yellowbrick_documents = cursor.fetchall()
print(f"成功提取了 {len(yellowbrick_documents)} 个文档！")
# 关闭游标和连接
cursor.close()
conn.close()
```

## 第四部分：使用文档加载 Yellowbrick 向量存储

遍历文档，将其拆分为可消化的块，创建嵌入并插入到 Yellowbrick 表中。这大约需要 5 分钟。

```python
# 将文档拆分为用于转换为嵌入的块
DOCUMENT_BASE_URL = "https://docs.yellowbrick.com/6.7.1/"  # 实际 URL
separator = "\n## "  # 此分隔符假定来自存储库的 Markdown 文档大部分时间使用 ### 作为逻辑主标题
chunk_size_limit = 2000
max_chunk_overlap = 200
documents = [
    Document(
        page_content=document[1],
        metadata={"source": DOCUMENT_BASE_URL + document[0].replace(".md", ".html")},
    )
    for document in yellowbrick_documents
]
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=chunk_size_limit,
    chunk_overlap=max_chunk_overlap,
    separators=[separator, "\nn", "\n", ",", " ", ""],
)
split_docs = text_splitter.split_documents(documents)
docs_text = [doc.page_content for doc in split_docs]
embeddings = OpenAIEmbeddings()
vector_store = Yellowbrick.from_documents(
    documents=split_docs,
    embedding=embeddings,
    connection_info=yellowbrick_connection_string,
    table=embedding_table,
)
print(f"使用 {len(documents)} 个文档创建了向量存储")
```

## 第五部分：创建使用 Yellowbrick 作为向量存储的聊天机器人

接下来，我们将 Yellowbrick 添加为向量存储。向量存储已经填充了代表 Yellowbrick 产品文档的管理章节的嵌入。

我们将发送与上述相同的查询，以查看改进后的响应。

```python
system_template = """使用以下上下文片段来回答用户的问题。
注意来源并以“来源1 来源2”的格式包含在答案中，使用大写的“来源”。
如果你不知道答案，只需说“我不知道”，不要试图编造答案。
----------------
{summaries}"""
messages = [
    SystemMessagePromptTemplate.from_template(system_template),
    HumanMessagePromptTemplate.from_template("{question}"),
]
prompt = ChatPromptTemplate.from_messages(messages)
vector_store = Yellowbrick(
    OpenAIEmbeddings(),
    yellowbrick_connection_string,
    embedding_table,  # 更改表名以反映您的嵌入
)
chain_type_kwargs = {"prompt": prompt}
llm = ChatOpenAI(
    model_name="gpt-3.5-turbo",  # 如果您可以访问 GPT-4，请修改 model_name
    temperature=0,
    max_tokens=256,
)
chain = RetrievalQAWithSourcesChain.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vector_store.as_retriever(search_kwargs={"k": 5}),
    return_source_documents=True,
    chain_type_kwargs=chain_type_kwargs,
)
def print_result_sources(query):
    result = chain(query)
    output_text = f"""### 问题: 
  {query}
  ### 答案: 
  {result['answer']}
  ### 来源: 
  {result['sources']}
  ### 所有相关来源:
  {', '.join(list(set([doc.metadata['source'] for doc in result['source_documents']])))}
    """
    display(Markdown(output_text))
# 使用链条进行查询
print_result_sources("一个 Yellowbrick 实例中可以有多少个数据库？")
print_result_sources("在 Yellowbrick 中批量添加用户的简单方法是什么？")
```

## 第六部分：引入索引以提高性能

Yellowbrick 还支持使用局部敏感哈希（Locality-Sensitive Hashing，LSH）方法进行索引。这是一种近似最近邻搜索技术，允许在准确性的代价上折衷相似性搜索时间。该索引引入了两个新的可调参数：

- 超平面的数量，作为参数提供给 `create_lsh_index(num_hyperplanes)`。文档越多，就需要更多的超平面。LSH 是一种降维技术。原始嵌入被转换为维数较低的向量，其中组件的数量与超平面的数量相同。

- 汉明距离，表示搜索的广度的整数。较小的汉明距离会导致更快的检索，但准确性较低。

以下是如何在我们加载到 Yellowbrick 中的嵌入上创建索引。我们还将重新运行之前的聊天会话，但这次检索将使用索引。请注意，对于如此少量的文档，您不会在性能方面看到索引的好处。

```python
system_template = """使用以下上下文片段来回答用户的问题。
注意来源并以以下格式将其包含在答案中：“来源：source1 source2”，不管来源数量如何，都使用大写的“来源”。
如果你不知道答案，只需说“我不知道”，不要试图凭空编造答案。
----------------
{summaries}"""
messages = [
    SystemMessagePromptTemplate.from_template(system_template),
    HumanMessagePromptTemplate.from_template("{question}"),
]
prompt = ChatPromptTemplate.from_messages(messages)
vector_store = Yellowbrick(
    OpenAIEmbeddings(),
    yellowbrick_connection_string,
    embedding_table,  # 更改表名以反映您的嵌入
)
lsh_params = Yellowbrick.IndexParams(
    Yellowbrick.IndexType.LSH, {"num_hyperplanes": 8, "hamming_distance": 2}
)
vector_store.create_index(lsh_params)
chain_type_kwargs = {"prompt": prompt}
llm = ChatOpenAI(
    model_name="gpt-3.5-turbo",  # 如果您可以访问 GPT-4，请修改 model_name
    temperature=0,
    max_tokens=256,
)
chain = RetrievalQAWithSourcesChain.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vector_store.as_retriever(
        k=5, search_kwargs={"index_params": lsh_params}
    ),
    return_source_documents=True,
    chain_type_kwargs=chain_type_kwargs,
)
def print_result_sources(query):
    result = chain(query)
    output_text = f"""### 问题： 
  {query}
  ### 答案： 
  {result['answer']}
  ### 来源： 
  {result['sources']}
  ### 所有相关来源：
  {', '.join(list(set([doc.metadata['source'] for doc in result['source_documents']])))}
    """
    display(Markdown(output_text))
# 使用链条进行查询
print_result_sources("一个 Yellowbrick 实例中可以有多少个数据库？")
print_result_sources("在 Yellowbrick 中批量添加用户的简单方法是什么？")
```

## 下一步：

可以修改此代码以提出不同的问题。您还可以将自己的文档加载到向量存储中。langchain 模块非常灵活，可以解析各种文件（包括 HTML、PDF 等）。

您还可以修改此代码，以使用 Huggingface 嵌入模型和 Meta 的 Llama 2 LLM，实现完全私密的聊天框体体验。