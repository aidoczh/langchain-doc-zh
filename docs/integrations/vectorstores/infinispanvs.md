# Infinispan

Infinispan是一个开源的键值数据网格，可以作为单节点和分布式工作。

自15.x版本开始支持向量搜索。

了解更多信息：[Infinispan主页](https://infinispan.org)

```python
# 确保我们安装了所有需要的内容
# 如果你愿意，可以跳过这一步
%pip install sentence-transformers
%pip install langchain
%pip install langchain_core
%pip install langchain_community
```

# 设置

为了运行这个演示，我们需要一个没有身份验证的运行中的Infinispan实例和一个数据文件。

在接下来的三个单元格中，我们将：

- 下载数据文件

- 创建配置文件

- 在Docker中运行Infinispan

```bash
%%bash
# 获取新闻的压缩文件
wget https://raw.githubusercontent.com/rigazilla/infinispan-vector/main/bbc_news.csv.gz
```

```bash
%%bash
# 创建Infinispan配置文件
echo 'infinispan:
  cache-container: 
    name: default
    transport: 
      cluster: cluster 
      stack: tcp 
  server:
    interfaces:
      interface:
        name: public
        inet-address:
          value: 0.0.0.0 
    socket-bindings:
      default-interface: public
      port-offset: 0        
      socket-binding:
        name: default
        port: 11222
    endpoints:
      endpoint:
        socket-binding: default
        rest-connector:
' > infinispan-noauth.yaml
```

```python
!docker rm --force infinispanvs-demo
!docker run -d --name infinispanvs-demo -v $(pwd):/user-config  -p 11222:11222 infinispan/server:15.0 -c /user-config/infinispan-noauth.yaml
```

# 代码

## 选择一个嵌入模型

在这个演示中，我们使用了HuggingFace的嵌入模型。

```python
from langchain.embeddings import HuggingFaceEmbeddings
from langchain_core.embeddings import Embeddings
model_name = "sentence-transformers/all-MiniLM-L12-v2"
hf = HuggingFaceEmbeddings(model_name=model_name)
```

## 设置Infinispan缓存

Infinispan是一个非常灵活的键值存储，它可以存储原始位和复杂的数据类型。

用户在数据网格配置方面拥有完全的自由，但对于简单的数据类型，Python层会自动配置一切。

我们利用这个特性，这样我们就可以专注于我们的应用程序。

## 准备数据

在这个演示中，我们依赖于默认配置，因此文本、元数据和向量都存储在同一个缓存中，但也可以选择其他选项：例如将内容存储在其他地方，向量存储只包含对实际内容的引用。

```python
import csv
import gzip
import time
# 打开新闻文件并将其处理为csv格式
with gzip.open("bbc_news.csv.gz", "rt", newline="") as csvfile:
    spamreader = csv.reader(csvfile, delimiter=",", quotechar='"')
    i = 0
    texts = []
    metas = []
    embeds = []
    for row in spamreader:
        # 将第一个和第五个值连接起来形成要处理的内容
        text = row[0] + "." + row[4]
        texts.append(text)
        # 将文本和标题存储为元数据
        meta = {"text": row[4], "title": row[0]}
        metas.append(meta)
        i = i + 1
        # 更改此处以更改要加载的新闻数量
        if i >= 5000:
            break
```

# 填充向量存储

```python
# 添加文本并填充向量数据库
from langchain_community.vectorstores import InfinispanVS
ispnvs = InfinispanVS.from_texts(texts, hf, metas)
```

# 一个打印结果文档的辅助函数

默认情况下，InfinispanVS将protobuf `ŧext`字段返回到`Document.page_content`中，将所有剩余的protobuf字段（向量除外）返回到`metadata`中。这个行为可以通过设置lambda函数来配置。

```python
def print_docs(docs):
    for res, i in zip(docs, range(len(docs))):
        print("----" + str(i + 1) + "----")
        print("TITLE: " + res.metadata["title"])
        print(res.page_content)
```

# 尝试一下!!!

以下是一些示例查询

```python
docs = ispnvs.similarity_search("European nations", 5)
print_docs(docs)
```

```python
print_docs(ispnvs.similarity_search("Milan fashion week begins", 2))
```

```python
print_docs(ispnvs.similarity_search("Stock market is rising today", 4))
```

```python
print_docs(ispnvs.similarity_search("Why cats are so viral?", 2))
```

```python
print_docs(ispnvs.similarity_search("How to stay young", 5))
```

```python
!docker rm --force infinispanvs-demo
```
