# Marqo

本文介绍了与 Marqo 向量存储相关的功能。

>[Marqo](https://www.marqo.ai/) 是一个开源的向量搜索引擎。Marqo 允许您存储和查询文本和图像等多模态数据。Marqo 使用大量的开源模型为您创建向量，您还可以提供自己的微调模型，Marqo 将为您处理加载和推理。

要使用我们的 Docker 镜像运行此笔记本，请首先运行以下命令获取 Marqo：

```
docker pull marqoai/marqo:latest
docker rm -f marqo
docker run --name marqo -it --privileged -p 8882:8882 --add-host host.docker.internal:host-gateway marqoai/marqo:latest
```
```python
%pip install --upgrade --quiet  marqo
```
```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Marqo
from langchain_text_splitters import CharacterTextSplitter
```
```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```
```python
import marqo
# 初始化 Marqo
marqo_url = "http://localhost:8882"  # 如果使用 Marqo 云端，请替换为您的端点 (console.marqo.ai)
marqo_api_key = ""  # 如果使用 Marqo 云端，请替换为您的 API 密钥 (console.marqo.ai)
client = marqo.Client(url=marqo_url, api_key=marqo_api_key)
index_name = "langchain-demo"
docsearch = Marqo.from_documents(docs, index_name=index_name)
query = "What did the president say about Ketanji Brown Jackson"
result_docs = docsearch.similarity_search(query)
```
```output
Index langchain-demo exists.
```
```python
print(result_docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```
```python
result_docs = docsearch.similarity_search_with_score(query)
print(result_docs[0][0].page_content, result_docs[0][1], sep="\n")
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
0.68647254
```

## 附加功能

作为向量存储的强大功能之一，Marqo 可以使用外部创建的索引。例如：

+ 如果您有来自其他应用程序的图像和文本对的数据库，您可以简单地在 langchain 中使用 Marqo 向量存储。请注意，使用自己的多模态索引将禁用 `add_texts` 方法。

+ 如果您有文本文档的数据库，您可以将其带入 langchain 框架，并通过 `add_texts` 添加更多文本。

通过将自己的函数传递给搜索方法中的 `page_content_builder` 回调，可以定制返回的文档。

#### 多模态示例

```python
# 使用一个新的索引
index_name = "langchain-multimodal-demo"
# 如果重新运行演示，则删除索引
try:
    client.delete_index(index_name)
except Exception:
    print(f"Creating {index_name}")
# 这个索引可以由另一个系统创建
settings = {"treat_urls_and_pointers_as_images": True, "model": "ViT-L/14"}
client.create_index(index_name, **settings)
client.index(index_name).add_documents(
    [
        # 一辆公交车的图像
        {
            "caption": "Bus",
            "image": "https://raw.githubusercontent.com/marqo-ai/marqo/mainline/examples/ImageSearchGuide/data/image4.jpg",
        },
        # 一架飞机的图像
        {
            "caption": "Plane",
            "image": "https://raw.githubusercontent.com/marqo-ai/marqo/mainline/examples/ImageSearchGuide/data/image2.jpg",
        },
    ],
)
```
```
{'errors': False,
 'processingTimeMs': 2090.2822139996715,
 'index_name': 'langchain-multimodal-demo',
 'items': [{'_id': 'aa92fc1c-1fb2-4d86-b027-feb507c419f7',
   'result': 'created',
   'status': 201},
  {'_id': '5142c258-ef9f-4bf2-a1a6-2307280173a0',
   'result': 'created',
   'status': 201}]}
```
```python
def get_content(res):
    """将 Marqo 的文档格式化为可用作页面内容的文本的辅助函数"""
    return f"{res['caption']}: {res['image']}"
docsearch = Marqo(client, index_name, page_content_builder=get_content)
query = "飞行器"
doc_results = docsearch.similarity_search(query)
```
```python
for doc in doc_results:
    print(doc.page_content)
```
```output
飞机: https://raw.githubusercontent.com/marqo-ai/marqo/mainline/examples/ImageSearchGuide/data/image2.jpg
公共汽车: https://raw.githubusercontent.com/marqo-ai/marqo/mainline/examples/ImageSearchGuide/data/image4.jpg
```

#### 仅文本示例

```python
# 使用新索引
index_name = "langchain-byo-index-demo"
# 如果重新运行演示
try:
    client.delete_index(index_name)
except Exception:
    print(f"创建 {index_name}")
# 这个索引可能已经被另一个系统创建
client.create_index(index_name)
client.index(index_name).add_documents(
    [
        {
            "Title": "智能手机",
            "Description": "智能手机是一种便携式计算机设备，将移动电话功能和计算功能合并为一个单元。",
        },
        {
            "Title": "电话",
            "Description": "电话是一种允许两个或多个用户进行对话的电信设备，当它们相隔太远而无法直接听到时。",
        },
    ],
)
```
```
{'errors': False,
 'processingTimeMs': 139.2144540004665,
 'index_name': 'langchain-byo-index-demo',
 'items': [{'_id': '27c05a1c-b8a9-49a5-ae73-fbf1eb51dc3f',
   'result': 'created',
   'status': 201},
  {'_id': '6889afe0-e600-43c1-aa3b-1d91bf6db274',
   'result': 'created',
   'status': 201}]}
```
```python
# 请注意，文本索引保留了使用 add_texts 的能力，尽管文档中的字段名称不同
# 这是因为 page_content_builder 回调函数允许您根据需要处理这些文档字段
def get_content(res):
    """将 Marqo 的文档格式化为可用作页面内容的文本的辅助函数"""
    if "text" in res:
        return res["text"]
    return res["Description"]
docsearch = Marqo(client, index_name, page_content_builder=get_content)
docsearch.add_texts(["这是一个关于大象的文档"])
```
```
['9986cc72-adcd-4080-9d74-265c173a9ec3']
```
```python
query = "现代通讯设备"
doc_results = docsearch.similarity_search(query)
print(doc_results[0].page_content)
```
```output
智能手机是一种便携式计算机设备，将移动电话功能和计算功能合并为一个单元。
```
```python
query = "大象"
doc_results = docsearch.similarity_search(query, page_content_builder=get_content)
print(doc_results[0].page_content)
```
```output
这是一个关于大象的文档
```

## 加权查询

我们还公开了 Marqo 的加权查询，这是一种组合复杂语义搜索的强大方式。

```python
query = {"通讯设备": 1.0}
doc_results = docsearch.similarity_search(query)
print(doc_results[0].page_content)
```
```output
智能手机是一种便携式计算机设备，将移动电话功能和计算功能合并为一个单元。
```
```python
query = {"通讯设备": 1.0, "2000年后的技术": -1.0}
doc_results = docsearch.similarity_search(query)
print(doc_results[0].page_content)
```
```output
电话是一种允许两个或多个用户进行对话的电信设备，当它们相隔太远而无法直接听到时。
```

# 带来源的问答

本节展示如何将 Marqo 作为 `RetrievalQAWithSourcesChain` 的一部分使用。Marqo 将在来源中执行信息搜索。

```python
import getpass
import os
from langchain.chains import RetrievalQAWithSourcesChain
from langchain_openai import OpenAI
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```
```output
OpenAI API Key:········
```
```python
with open("../../how_to/state_of_the_union.txt") as f:
    state_of_the_union = f.read()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_text(state_of_the_union)
```
```python
index_name = "langchain-qa-with-retrieval"
docsearch = Marqo.from_documents(docs, index_name=index_name)
```
```output
索引 langchain-qa-with-retrieval 已存在。
```
```python
chain = RetrievalQAWithSourcesChain.from_chain_type(
```python

OpenAI(temperature=0), chain_type="stuff", retriever=docsearch.as_retriever()

)

```
```python
chain(
    {"question": "总统对布雷耶大法官说了什么"},
    return_only_outputs=True,
)
```
```output

{'answer': '总统向布雷耶大法官致敬，感谢他的服务，并指出他是美国最高法院的一名即将退休的大法官。\n',

 'sources': '../../../state_of_the_union.txt'}

```