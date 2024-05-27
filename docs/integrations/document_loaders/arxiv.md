# Arxiv

>[arXiv](https://arxiv.org/) 是一个开放获取的存档，收录了来自物理学、数学、计算机科学、数量生物学、数量金融、统计学、电气工程与系统科学以及经济学等领域的 200 万篇学术文章。

本文档展示了如何从 `Arxiv.org` 加载科学文章，并将其转换为我们可以在下游使用的文档格式。

## 安装

首先，您需要安装 `arxiv` python 包。

```python
%pip install --upgrade --quiet  arxiv
```

其次，您需要安装 `PyMuPDF` python 包，该包可以将从 `arxiv.org` 网站下载的 PDF 文件转换为文本格式。

```python
%pip install --upgrade --quiet  pymupdf
```

## 示例

`ArxivLoader` 有以下参数：

- `query`: 用于在 Arxiv 中查找文档的自由文本

- 可选参数 `load_max_docs`: 默认值为 100。使用它来限制下载文档的数量。下载所有 100 份文件需要时间，因此在实验中使用较小的数字。

- 可选参数 `load_all_available_meta`: 默认值为 False。默认情况下，只下载最重要的字段：`Published`（文档发布/最后更新日期）、`Title`（标题）、`Authors`（作者）、`Summary`（摘要）。如果为 True，则还会下载其他字段。

```python
from langchain_community.document_loaders import ArxivLoader
```

```python
docs = ArxivLoader(query="1605.08386", load_max_docs=2).load()
len(docs)
```

```python
docs[0].metadata  # 文档的元信息
```

```output
{'Published': '2016-05-26',
 'Title': 'Heat-bath random walks with Markov bases',
 'Authors': 'Caprice Stanley, Tobias Windisch',
 'Summary': 'Graphs on lattice points are studied whose edges come from a finite set of\nallowed moves of arbitrary length. We show that the diameter of these graphs on\nfibers of a fixed integer matrix can be bounded from above by a constant. We\nthen study the mixing behaviour of heat-bath random walks on these graphs. We\nalso state explicit conditions on the set of moves so that the heat-bath random\nwalk, a generalization of the Glauber dynamics, is an expander in fixed\ndimension.'}
```

```python
docs[0].page_content[:400]  # 文档内容的所有页面
```

```output
'arXiv:1605.08386v1  [math.CO]  26 May 2016\nHEAT-BATH RANDOM WALKS WITH MARKOV BASES\nCAPRICE STANLEY AND TOBIAS WINDISCH\nAbstract. Graphs on lattice points are studied whose edges come from a ﬁnite set of\nallowed moves of arbitrary length. We show that the diameter of these graphs on ﬁbers of a\nﬁxed integer matrix can be bounded from above by a constant. We then study the mixing\nbehaviour of heat-b'
```