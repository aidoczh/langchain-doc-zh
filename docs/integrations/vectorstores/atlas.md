# Atlas

[Atlas](https://docs.nomic.ai/index.html) 是 Nomic 公司开发的一个平台，用于与小型和互联网规模的非结构化数据集进行交互。它使任何人都能够在浏览器中可视化、搜索和共享大规模数据集。

本文档向您展示了如何使用与 `AtlasDB` 向量存储相关的功能。

```python
%pip install --upgrade --quiet  spacy
```

```python
!python3 -m spacy download en_core_web_sm
```

```python
%pip install --upgrade --quiet  nomic
```

### 加载包

```python
import time
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import AtlasDB
from langchain_text_splitters import SpacyTextSplitter
```

```python
ATLAS_TEST_API_KEY = "7xDPkYXSYDc1_ErdTPIcoAR9RNd8YDlkS3nVNXcVoIMZ6"
```

### 准备数据

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = SpacyTextSplitter(separator="|")
texts = []
for doc in text_splitter.split_documents(documents):
    texts.extend(doc.page_content.split("|"))
texts = [e.strip() for e in texts]
```

### 使用 Nomic 的 Atlas 进行数据映射

```python
db = AtlasDB.from_texts(
    texts=texts,
    name="test_index_" + str(time.time()),  # 为您的向量存储指定一个唯一的名称
    description="test_index",  # 为您的向量存储指定一个描述
    api_key=ATLAS_TEST_API_KEY,
    index_kwargs={"build_topic_model": True},
)
```

```python
db.project.wait_for_project_lock()
```

```python
db.project
```

这是代码的结果地图。该地图显示了《国情咨文》的文本。

https://atlas.nomic.ai/map/3e4de075-89ff-486a-845c-36c23f30bb67/d8ce2284-8edb-4050-8b9b-9bb543d7f647