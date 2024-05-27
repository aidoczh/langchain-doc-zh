# TF-IDF

[TF-IDF](https://scikit-learn.org/stable/modules/feature_extraction.html#tfidf-term-weighting) 意为词项频率乘以逆文档频率。

本笔记介绍了如何使用一个底层使用 [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) 的检索器，使用了 `scikit-learn` 软件包。

有关 TF-IDF 详细信息，请参阅[此博文](https://medium.com/data-science-bootcamp/tf-idf-basics-of-information-retrieval-48de122b2a4c)。

```python
%pip install --upgrade --quiet  scikit-learn
```

```python
from langchain_community.retrievers import TFIDFRetriever
```

## 使用文本创建新的检索器

```python
retriever = TFIDFRetriever.from_texts(["foo", "bar", "world", "hello", "foo bar"])
```

## 使用文档创建新的检索器

现在，您可以使用创建的文档创建一个新的检索器。

```python
from langchain_core.documents import Document
retriever = TFIDFRetriever.from_documents(
    [
        Document(page_content="foo"),
        Document(page_content="bar"),
        Document(page_content="world"),
        Document(page_content="hello"),
        Document(page_content="foo bar"),
    ]
)
```

## 使用检索器

现在我们可以使用检索器了！

```python
result = retriever.invoke("foo")
```

```python
result
```

```output
[Document(page_content='foo', metadata={}),
 Document(page_content='foo bar', metadata={}),
 Document(page_content='hello', metadata={}),
 Document(page_content='world', metadata={})]
```

## 保存和加载

您可以轻松保存和加载这个检索器，使其非常适合本地开发！

```python
retriever.save_local("testing.pkl")
```

```python
retriever_copy = TFIDFRetriever.load_local("testing.pkl")
```

```python
retriever_copy.invoke("foo")
```

```output
[Document(page_content='foo', metadata={}),
 Document(page_content='foo bar', metadata={}),
 Document(page_content='hello', metadata={}),
 Document(page_content='world', metadata={})]
```