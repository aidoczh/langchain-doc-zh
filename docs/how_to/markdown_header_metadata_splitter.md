# 如何通过标题拆分 Markdown

### 动机

许多聊天或问答应用程序在嵌入和向量存储之前需要对输入文档进行分块。

[Pinecone的这些笔记](https://www.pinecone.io/learn/chunking-strategies/)提供了一些有用的提示：

```
当嵌入整个段落或文档时，嵌入过程会考虑文本中句子和短语之间的整体上下文和关系。这可能会导致更全面的向量表示，捕捉到文本的更广泛的含义和主题。
```

正如上面提到的，分块通常旨在将具有共同上下文的文本保持在一起。考虑到这一点，我们可能希望特别尊重文档本身的结构。例如，Markdown文件是按标题组织的。在特定标题组中创建分块是一个直观的想法。为了解决这个挑战，我们可以使用[MarkdownHeaderTextSplitter](https://api.python.langchain.com/en/latest/markdown/langchain_text_splitters.markdown.MarkdownHeaderTextSplitter.html)。它可以根据指定的一组标题来拆分Markdown文件。

例如，如果我们想要拆分这个Markdown：

```
md = '# Foo\n\n ## Bar\n\nHi this is Jim  \nHi this is Joe\n\n ## Baz\n\n Hi this is Molly' 
```

我们可以指定要拆分的标题：

```
[("#", "Header 1"),("##", "Header 2")]
```

内容将根据共同的标题进行分组或拆分：

```
{'content': 'Hi this is Jim  \nHi this is Joe', 'metadata': {'Header 1': 'Foo', 'Header 2': 'Bar'}}
{'content': 'Hi this is Molly', 'metadata': {'Header 1': 'Foo', 'Header 2': 'Baz'}}
```

让我们看一些下面的示例。

### 基本用法：

```python
%pip install -qU langchain-text-splitters
```

```python
from langchain_text_splitters import MarkdownHeaderTextSplitter
```

```python
markdown_document = "# Foo\n\n    ## Bar\n\nHi this is Jim\n\nHi this is Joe\n\n ### Boo \n\n Hi this is Lance \n\n ## Baz\n\n Hi this is Molly"
headers_to_split_on = [
    ("#", "Header 1"),
    ("##", "Header 2"),
    ("###", "Header 3"),
]
markdown_splitter = MarkdownHeaderTextSplitter(headers_to_split_on)
md_header_splits = markdown_splitter.split_text(markdown_document)
md_header_splits
```

```output
[Document(page_content='Hi this is Jim  \nHi this is Joe', metadata={'Header 1': 'Foo', 'Header 2': 'Bar'}),
 Document(page_content='Hi this is Lance', metadata={'Header 1': 'Foo', 'Header 2': 'Bar', 'Header 3': 'Boo'}),
 Document(page_content='Hi this is Molly', metadata={'Header 1': 'Foo', 'Header 2': 'Baz'})]
```

```python
type(md_header_splits[0])
```

```output
langchain_core.documents.base.Document
```

默认情况下，`MarkdownHeaderTextSplitter`会从输出块的内容中删除正在拆分的标题。可以通过设置`strip_headers = False`来禁用此功能。

```python
markdown_splitter = MarkdownHeaderTextSplitter(headers_to_split_on, strip_headers=False)
md_header_splits = markdown_splitter.split_text(markdown_document)
md_header_splits
```

```output
[Document(page_content='# Foo  \n## Bar  \nHi this is Jim  \nHi this is Joe', metadata={'Header 1': 'Foo', 'Header 2': 'Bar'}),
 Document(page_content='### Boo  \nHi this is Lance', metadata={'Header 1': 'Foo', 'Header 2': 'Bar', 'Header 3': 'Boo'}),
 Document(page_content='## Baz  \nHi this is Molly', metadata={'Header 1': 'Foo', 'Header 2': 'Baz'})]
```

### 如何将Markdown行作为单独的文档返回

默认情况下，`MarkdownHeaderTextSplitter`根据`headers_to_split_on`中指定的标题聚合行。我们可以通过指定`return_each_line`来禁用此功能：

```python
markdown_splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on,
    return_each_line=True,
)
md_header_splits = markdown_splitter.split_text(markdown_document)
md_header_splits
```

```output
[Document(page_content='Hi this is Jim', metadata={'Header 1': 'Foo', 'Header 2': 'Bar'}),
 Document(page_content='Hi this is Joe', metadata={'Header 1': 'Foo', 'Header 2': 'Bar'}),
 Document(page_content='Hi this is Lance', metadata={'Header 1': 'Foo', 'Header 2': 'Bar', 'Header 3': 'Boo'}),
 Document(page_content='Hi this is Molly', metadata={'Header 1': 'Foo', 'Header 2': 'Baz'})]
```

请注意，这里的标题信息在每个文档的`metadata`中保留。

### 如何限制块大小

在每个Markdown组中，我们可以应用任何我们想要的文本分割器，例如`RecursiveCharacterTextSplitter`，它允许进一步控制块的大小。

```python
markdown_document = "# Intro \n\n    ## History \n\n Markdown[9] is a lightweight markup language for creating formatted text using a plain-text editor. John Gruber created Markdown in 2004 as a markup language that is appealing to human readers in its source code form.[9] \n\n Markdown is widely used in blogging, instant messaging, online forums, collaborative software, documentation pages, and readme files. \n\n ## Rise and divergence \n\n As Markdown popularity grew rapidly, many Markdown implementations appeared, driven mostly by the need for \n\n additional features such as tables, footnotes, definition lists,[note 1] and Markdown inside HTML blocks. \n\n #### Standardization \n\n From 2012, a group of people, including Jeff Atwood and John MacFarlane, launched what Atwood characterised as a standardisation effort. \n\n ## Implementations \n\n Implementations of Markdown are available for over a dozen programming languages."
headers_to_split_on = [
    ("#", "Header 1"),
    ("##", "Header 2"),
]
markdown_splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=headers_to_split_on, strip_headers=False
)
md_header_splits = markdown_splitter.split_text(markdown_document)
from langchain_text_splitters import RecursiveCharacterTextSplitter
chunk_size = 250
chunk_overlap = 30
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=chunk_size, chunk_overlap=chunk_overlap
)
splits = text_splitter.split_documents(md_header_splits)
splits
```

# 简介

## 历史

Markdown[9] 是一种轻量级标记语言，可用于使用纯文本编辑器创建格式化文本。John Gruber 在 2004 年创建了 Markdown，作为一种在源代码形式中吸引人类读者的标记语言[9]。

Markdown 在博客、即时通讯、在线论坛、协作软件、文档页面和自述文件中被广泛使用。

## 兴起与分歧

随着 Markdown 的流行迅速增长，出现了许多 Markdown 实现，主要是出于对表格、脚注、定义列表[note 1]和 HTML 块内的 Markdown 等附加功能的需求。

#### 标准化

从 2012 年开始，包括 Jeff Atwood 和 John MacFarlane 在内的一群人发起了 Atwood 所描述的标准化工作。

## 实现

Markdown 的实现可用于十多种编程语言。