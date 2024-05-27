# ScaNN

ScaNN（Scalable Nearest Neighbors）是一种用于大规模高效向量相似度搜索的方法。

ScaNN包括搜索空间修剪和最大内积搜索的量化，同时支持其他距离函数，如欧氏距离。该实现针对具有AVX2支持的x86处理器进行了优化。更多详情请参见其[Google Research github](https://github.com/google-research/google-research/tree/master/scann)。

## 安装

通过pip安装ScaNN。或者，您可以按照[ScaNN网站](https://github.com/google-research/google-research/tree/master/scann#building-from-source)上的说明从源代码安装。

```python
%pip install --upgrade --quiet  scann
```

## 检索演示

下面我们将展示如何将ScaNN与Huggingface Embeddings结合使用。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import ScaNN
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = HuggingFaceEmbeddings()
db = ScaNN.from_documents(docs, embeddings)
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
docs[0]
```

```output
Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': 'state_of_the_union.txt'})
```

## RetrievalQA 演示

接下来，我们演示如何将ScaNN与Google PaLM API结合使用。

您可以从https://developers.generativeai.google/tutorials/setup获取API密钥

```python
from langchain.chains import RetrievalQA
from langchain_community.chat_models import google_palm
palm_client = google_palm.ChatGooglePalm(google_api_key="YOUR_GOOGLE_PALM_API_KEY")
qa = RetrievalQA.from_chain_type(
    llm=palm_client,
    chain_type="stuff",
    retriever=db.as_retriever(search_kwargs={"k": 10}),
)
```

```python
print(qa.run("What did the president say about Ketanji Brown Jackson?"))
```

```output
The president said that Ketanji Brown Jackson is one of our nation's top legal minds, who will continue Justice Breyer's legacy of excellence.
```

```python
print(qa.run("What did the president say about Michael Phelps?"))
```

```output
The president did not mention Michael Phelps in his speech.
```

## 保存和加载本地检索索引

```python
db.save_local("/tmp/db", "state_of_union")
restored_db = ScaNN.load_local("/tmp/db", embeddings, index_name="state_of_union")
```