# StarRocks
[StarRocks](https://www.starrocks.io/) 是一款高性能的分析型数据库。
`StarRocks` 是一款面向全面分析场景的下一代亚秒级 MPP 数据库，包括多维分析、实时分析和即席查询。
通常情况下，`StarRocks` 被归类为 OLAP，它在[ClickBench — a Benchmark For Analytical DBMS](https://benchmark.clickhouse.com/)中表现出色。由于它拥有超快的矢量化执行引擎，因此也可以用作快速的向量数据库。
下面我们将展示如何使用 StarRocks Vector Store。
## 设置
```python
%pip install --upgrade --quiet  pymysql
```
在开始时将 `update_vectordb = False`。如果没有更新文档，那么我们就不需要重建文档的嵌入。
```python
from langchain.chains import RetrievalQA
from langchain_community.document_loaders import (
    DirectoryLoader,
    UnstructuredMarkdownLoader,
)
from langchain_community.vectorstores import StarRocks
from langchain_community.vectorstores.starrocks import StarRocksSettings
from langchain_openai import OpenAI, OpenAIEmbeddings
from langchain_text_splitters import TokenTextSplitter
update_vectordb = False
```
```output
/Users/dirlt/utils/py3env/lib/python3.9/site-packages/requests/__init__.py:102: RequestsDependencyWarning: urllib3 (1.26.7) or chardet (5.1.0)/charset_normalizer (2.0.9) doesn't match a supported version!
  warnings.warn("urllib3 ({}) or chardet ({})/charset_normalizer ({}) doesn't match a supported "
```
## 加载文档并将其拆分为标记
加载 `docs` 目录下的所有 markdown 文件。
对于 starrocks 文档，您可以从 https://github.com/StarRocks/starrocks 克隆存储库，在其中有一个 `docs` 目录。
```python
loader = DirectoryLoader(
    "./docs", glob="**/*.md", loader_cls=UnstructuredMarkdownLoader
)
documents = loader.load()
```
将文档拆分为标记，并设置 `update_vectordb = True`，因为有新的文档/标记。
```python
# 加载文本拆分器并将文档拆分为文本片段
text_splitter = TokenTextSplitter(chunk_size=400, chunk_overlap=50)
split_docs = text_splitter.split_documents(documents)
# 告诉向量数据库更新文本嵌入
update_vectordb = True
```
```python
split_docs[-20]
```
```output
Document(page_content='Compile StarRocks with Docker\n\nThis topic describes how to compile StarRocks using Docker.\n\nOverview\n\nStarRocks provides development environment images for both Ubuntu 22.04 and CentOS 7.9. With the image, you can launch a Docker container and compile StarRocks in the container.\n\nStarRocks version and DEV ENV image\n\nDifferent branches of StarRocks correspond to different development environment images provided on StarRocks Docker Hub.\n\nFor Ubuntu 22.04:\n\n| Branch name | Image name              |\n  | --------------- | ----------------------------------- |\n  | main            | starrocks/dev-env-ubuntu:latest     |\n  | branch-3.0      | starrocks/dev-env-ubuntu:3.0-latest |\n  | branch-2.5      | starrocks/dev-env-ubuntu:2.5-latest |\n\nFor CentOS 7.9:\n\n| Branch name | Image name                       |\n  | --------------- | ------------------------------------ |\n  | main            | starrocks/dev-env-centos7:latest     |\n  | branch-3.0      | starrocks/dev-env-centos7:3.0-latest |\n  | branch-2.5      | starrocks/dev-env-centos7:2.5-latest |\n\nPrerequisites\n\nBefore compiling StarRocks, make sure the following requirements are satisfied:\n\nHardware\n\n', metadata={'source': 'docs/developers/build-starrocks/Build_in_docker.md'})
```
```python
print("# docs  = %d, # splits = %d" % (len(documents), len(split_docs)))
```
```output
# docs  = 657, # splits = 2802
```
## 创建向量数据库实例
### 使用 StarRocks 作为向量数据库
```python
def gen_starrocks(update_vectordb, embeddings, settings):
    if update_vectordb:
        docsearch = StarRocks.from_documents(split_docs, embeddings, config=settings)
    else:
        docsearch = StarRocks(embeddings, settings)
    return docsearch
```
## 将标记转换为嵌入并将其放入向量数据库
在这里，我们使用 StarRocks 作为向量数据库，您可以通过 `StarRocksSettings` 配置 StarRocks 实例。
配置 StarRocks 实例与配置 mysql 实例非常相似。您需要指定：
1. 主机/端口
2. 用户名（默认值：'root'）
3. 密码（默认值：''）
4. 数据库（默认值：'default'）
5. 表（默认值：'langchain'）
```python
embeddings = OpenAIEmbeddings()
# 配置 starrocks 设置（主机/端口/用户/密码/数据库）
settings = StarRocksSettings()
settings.port = 41003
settings.host = "127.0.0.1"
settings.username = "root"
settings.password = ""
settings.database = "zya"
docsearch = gen_starrocks(update_vectordb, embeddings, settings)
print(docsearch)
update_vectordb = False
```
```output
Inserting data...: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2802/2802 [02:26<00:00, 19.11it/s]
```
```output
zya.langchain @ 127.0.0.1:41003
username: root
Table Schema:
----------------------------------------------------------------------------
|name                    |type                    |key                     |
----------------------------------------------------------------------------
|id                      |varchar(65533)          |true                    ||document                |varchar(65533)          |false                   ||embedding               |array<float>            |false                   ||metadata                |varchar(65533)          |false                   |
----------------------------------------------------------------------------
```
## 构建问答系统并向其提问
```python
llm = OpenAI()
qa = RetrievalQA.from_chain_type(
    llm=llm, chain_type="stuff", retriever=docsearch.as_retriever()
)
query = "is profile enabled by default? if not, how to enable profile?"
resp = qa.run(query)
print(resp)
```
```output
 不，配置文件默认情况下未启用。要启用配置文件，请使用命令 `set enable_profile = true;` 将变量 `enable_profile` 设置为 `true`。
```