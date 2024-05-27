# 支持向量机（SVM）

[支持向量机（SVM）](https://scikit-learn.org/stable/modules/svm.html#support-vector-machines)是一组用于分类、回归和异常值检测的监督学习方法。

本笔记介绍了如何使用一个底层使用 `scikit-learn` 包的 `SVM` 的检索器。

主要基于 https://github.com/karpathy/randomfun/blob/master/knn_vs_svm.html

```python
%pip install --upgrade --quiet  scikit-learn
```

```python
%pip install --upgrade --quiet  lark
```

我们想要使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```output
OpenAI API Key: ········
```

```python
from langchain_community.retrievers import SVMRetriever
from langchain_openai import OpenAIEmbeddings
```

## 创建新的检索器与文本

```python
retriever = SVMRetriever.from_texts(
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
 Document(page_content='world', metadata={})]
```