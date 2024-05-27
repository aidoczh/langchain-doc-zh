# Google Firestore（原生模式）
> [Firestore](https://cloud.google.com/firestore) 是一个无服务器的面向文档的数据库，可以根据需求进行扩展。通过使用Firestore的Langchain集成，扩展您的数据库应用程序以构建基于人工智能的体验。
本笔记本介绍了如何使用[Firestore](https://cloud.google.com/firestore)和`FirestoreLoader`以及`FirestoreSaver`来[保存、加载和删除langchain文档](/docs/how_to#document-loaders)。
在[GitHub](https://github.com/googleapis/langchain-google-firestore-python/)上了解有关该软件包的更多信息。
[![在Colab中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-firestore-python/blob/main/docs/document_loader.ipynb)
## 开始之前
要运行此笔记本，您需要执行以下操作：
- [创建一个Google Cloud项目](https://developers.google.com/workspace/guides/create-project)
- [启用Firestore API](https://console.cloud.google.com/flows/enableapi?apiid=firestore.googleapis.com)
- [创建一个Firestore数据库](https://cloud.google.com/firestore/docs/manage-databases)
在确认在此笔记本的运行时环境中可以访问数据库后，填写以下值并在运行示例脚本之前运行单元格。
```python
# @markdown 请指定一个用于演示目的的源。
SOURCE = "test"  # @param {type:"Query"|"CollectionGroup"|"DocumentReference"|"string"}
```
### 🦜🔗 安装库
该集成位于其自己的`langchain-google-firestore`软件包中，因此我们需要安装它。
```python
%pip install -upgrade --quiet langchain-google-firestore
```
**仅限Colab**：取消注释以下单元格以重新启动内核，或使用顶部的按钮重新启动内核。对于Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。
```python
# # 在安装后自动重新启动内核，以便您的环境可以访问新的软件包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```
### ☁ 设置您的Google Cloud项目
设置您的Google Cloud项目，以便您可以在此笔记本中利用Google Cloud资源。
如果您不知道项目ID，请尝试以下操作：
- 运行`gcloud config list`。
- 运行`gcloud projects list`。
- 参见支持页面：[查找项目ID](https://support.google.com/googleapi/answer/7014113)。
```python
# @markdown 请在下面的值中填写您的Google Cloud项目ID，然后运行该单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目ID
!gcloud config set project {PROJECT_ID}
```
### 🔐 身份验证
以IAM用户的身份登录到此笔记本中的Google Cloud，以便访问您的Google Cloud项目。
- 如果您使用Colab运行此笔记本，请使用下面的单元格并继续。
- 如果您使用的是Vertex AI Workbench，请查看此处的设置说明[here](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)。
```python
from google.colab import auth
auth.authenticate_user()
```
## 基本用法
### 保存文档
`FirestoreSaver`可以将文档存储到Firestore中。默认情况下，它将尝试从元数据中提取文档引用。
使用`FirestoreSaver.upsert_documents(<documents>)`保存langchain文档。
```python
from langchain_core.documents import Document
from langchain_google_firestore import FirestoreSaver
saver = FirestoreSaver()
data = [Document(page_content="Hello, World!")]
saver.upsert_documents(data)
```
#### 保存没有引用的文档
如果指定了一个集合，文档将以自动生成的ID存储。
```python
saver = FirestoreSaver("Collection")
saver.upsert_documents(data)
```
#### 保存具有其他引用的文档
```python
doc_ids = ["AnotherCollection/doc_id", "foo/bar"]
saver = FirestoreSaver()
saver.upsert_documents(documents=data, document_ids=doc_ids)
```
### 从集合或子集合加载
使用`FirestoreLoader.load()`或`Firestore.lazy_load()`加载langchain文档。`lazy_load`返回一个只在迭代期间查询数据库的生成器。要初始化`FirestoreLoader`类，您需要提供：
1. `source` - Query、CollectionGroup、DocumentReference的实例或指向Firestore集合的单个`\`分隔路径。
```python
from langchain_google_firestore import FirestoreLoader
loader_collection = FirestoreLoader("Collection")
loader_subcollection = FirestoreLoader("Collection/doc/SubCollection")
data_collection = loader_collection.load()
data_subcollection = loader_subcollection.load()
```
### 加载单个文档
```python
from google.cloud import firestore
client = firestore.Client()
doc_ref = client.collection("foo").document("bar")
loader_document = FirestoreLoader(doc_ref)
data = loader_document.load()
```
### 从 CollectionGroup 或 Query 加载数据
```python
from google.cloud.firestore import CollectionGroup, FieldFilter, Query
col_ref = client.collection("col_group")
collection_group = CollectionGroup(col_ref)
loader_group = FirestoreLoader(collection_group)
col_ref = client.collection("collection")
query = col_ref.where(filter=FieldFilter("region", "==", "west_coast"))
loader_query = FirestoreLoader(query)
```
### 删除文档
使用 `FirestoreSaver.delete_documents(<documents>)` 从 Firestore 集合中删除一组 langchain 文档。
如果提供了文档 id，则会忽略文档内容。
```python
saver = FirestoreSaver()
saver.delete_documents(data)
# 仅使用文档 id，忽略文档内容。
saver.delete_documents(data, doc_ids)
```
## 高级用法
### 自定义文档页面内容和元数据加载
`page_content_fields` 和 `metadata_fields` 的参数将指定要写入 LangChain 文档的 Firestore 文档字段 `page_content` 和 `metadata`。
```python
loader = FirestoreLoader(
    source="foo/bar/subcol",
    page_content_fields=["data_field"],
    metadata_fields=["metadata_field"],
)
data = loader.load()
```
#### 自定义页面内容格式
当 `page_content` 仅包含一个字段时，信息将仅为该字段的值。否则，`page_content` 将以 JSON 格式呈现。
### 自定义连接和身份验证
```python
from google.auth import compute_engine
from google.cloud.firestore import Client
client = Client(database="non-default-db", creds=compute_engine.Credentials())
loader = FirestoreLoader(
    source="foo",
    client=client,
)
```
```