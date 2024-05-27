# kNN

在统计学中，[k-最近邻算法 (k-NN)](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm) 是一种非参数的监督学习方法，最早由 `Evelyn Fix` 和 `Joseph Hodges` 在 1951 年开发，后来由 `Thomas Cover` 进行了扩展。它被用于分类和回归。

本文档介绍了如何使用底层使用 kNN 的检索器。

主要基于 [Andrej Karpathy](https://github.com/karpathy/randomfun/blob/master/knn_vs_svm.html) 的代码。

```python
from langchain_community.retrievers import KNNRetriever
from langchain_openai import OpenAIEmbeddings
```

## 创建带有文本的新检索器

```python
retriever = KNNRetriever.from_texts(
    ["foo", "bar", "world", "hello", "foo bar"], OpenAIEmbeddings()
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
 Document(page_content='bar', metadata={})]
```