# 维基百科

[维基百科](https://wikipedia.org/)是一个由志愿者社区（称为维基人）撰写和维护的多语言免费在线百科全书，通过开放协作并使用名为MediaWiki的基于维基的编辑系统。`维基百科`是历史上最大且阅读量最高的参考作品。

这篇笔记展示了如何从`wikipedia.org`加载维基页面到我们下游使用的文档格式。

## 安装

首先，您需要安装`wikipedia` python包。

```python
%pip install --upgrade --quiet  wikipedia
```

## 示例

`WikipediaLoader`有以下参数：

- `query`: 用于在维基百科中查找文档的自由文本

- 可选 `lang`: 默认值为"en"。用于在特定语言部分的维基百科中进行搜索

- 可选 `load_max_docs`: 默认值为100。用于限制下载文档的数量。下载所有100个文档需要时间，因此在实验中使用较小的数字。目前有一个硬性限制为300。

- 可选 `load_all_available_meta`: 默认值为False。默认情况下，仅下载最重要的字段：`Published`（文档发布/最后更新日期）、`title`、`Summary`。如果为True，则还会下载其他字段。

```python
from langchain_community.document_loaders import WikipediaLoader
```

```python
docs = WikipediaLoader(query="HUNTER X HUNTER", load_max_docs=2).load()
len(docs)
```

```python
docs[0].metadata  # 文档的元信息
```

```python
docs[0].page_content[:400]  # 文档的内容
```