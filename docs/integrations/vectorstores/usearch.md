# USearch

[USearch](https://unum-cloud.github.io/usearch/) 是一个更小更快的单文件向量搜索引擎

USearch 的基本功能与 FAISS 相同，如果你曾经调查过近似最近邻搜索，那么它的界面应该很熟悉。FAISS 是高性能向量搜索引擎的广泛认可标准。USearch 和 FAISS 都采用相同的 HNSW 算法，但它们在设计原则上有显著的不同。USearch 紧凑且广泛兼容，而且不会牺牲性能，主要关注用户定义的度量标准，并减少了依赖关系。

```python
%pip install --upgrade --quiet  usearch
```

我们想要使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import USearch
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../../extras/modules/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

```python
db = USearch.from_documents(docs, embeddings)
query = "总统对Ketanji Brown Jackson有什么看法"
docs = db.similarity_search(query)
```

```python
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法案》。而且，趁机通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶法官，感谢您的服务。
总统的最重要的宪法责任之一就是提名某人担任美国最高法院的法官。
而我在4天前就做到了，当时我提名了联邦上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续延续布雷耶法官的卓越传统。
```

## 带有分数的相似性搜索

`similarity_search_with_score` 方法允许您不仅返回文档，还返回查询与它们之间的距离分数。返回的距离分数是 L2 距离。因此，得分越低越好。

```python
docs_and_scores = db.similarity_search_with_score(query)
```

```python
docs_and_scores[0]
```

```output
(Document(page_content='今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法案》。而且，趁机通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶法官，感谢您的服务。\n\n总统的最重要的宪法责任之一就是提名某人担任美国最高法院的法官。\n\n而我在4天前就做到了，当时我提名了联邦上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续延续布雷耶法官的卓越传统。', metadata={'source': '../../../extras/modules/state_of_the_union.txt'}),
 0.1845687)
```