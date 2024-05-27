# 如何按照 HTML 标题进行分割

## 描述和动机

[HTMLHeaderTextSplitter](https://api.python.langchain.com/en/latest/html/langchain_text_splitters.html.HTMLHeaderTextSplitter.html) 是一个“结构感知”的分块器，它可以在 HTML 元素级别上对文本进行分割，并为每个与给定分块相关的标题添加元数据。它可以逐个元素返回分块，也可以将具有相同元数据的元素组合在一起，其目标是(a)将相关文本（或多或少）语义地分组，以及(b)保留在文档结构中编码的上下文丰富的信息。它可以作为分块流水线的一部分与其他文本分割器一起使用。

它类似于用于 markdown 文件的 [MarkdownHeaderTextSplitter](/docs/how_to/markdown_header_metadata_splitter)。

要指定要分割的标题，请在实例化 `HTMLHeaderTextSplitter` 时指定 `headers_to_split_on`，如下所示。

## 使用示例

### 1) 如何分割 HTML 字符串：

```python
%pip install -qU langchain-text-splitters
```

```python
from langchain_text_splitters import HTMLHeaderTextSplitter
html_string = """
<!DOCTYPE html>
<html>
<body>
    <div>
        <h1>Foo</h1>
        <p>Some intro text about Foo.</p>
        <div>
            <h2>Bar main section</h2>
            <p>Some intro text about Bar.</p>
            <h3>Bar subsection 1</h3>
            <p>Some text about the first subtopic of Bar.</p>
            <h3>Bar subsection 2</h3>
            <p>Some text about the second subtopic of Bar.</p>
        </div>
        <div>
            <h2>Baz</h2>
            <p>Some text about Baz</p>
        </div>
        <br>
        <p>Some concluding text about Foo</p>
    </div>
</body>
</html>
"""
headers_to_split_on = [
    ("h1", "Header 1"),
    ("h2", "Header 2"),
    ("h3", "Header 3"),
]
html_splitter = HTMLHeaderTextSplitter(headers_to_split_on)
html_header_splits = html_splitter.split_text(html_string)
html_header_splits
```

```output
[Document(page_content='Foo'),
 Document(page_content='Some intro text about Foo.  \nBar main section Bar subsection 1 Bar subsection 2', metadata={'Header 1': 'Foo'}),
 Document(page_content='Some intro text about Bar.', metadata={'Header 1': 'Foo', 'Header 2': 'Bar main section'}),
 Document(page_content='Some text about the first subtopic of Bar.', metadata={'Header 1': 'Foo', 'Header 2': 'Bar main section', 'Header 3': 'Bar subsection 1'}),
 Document(page_content='Some text about the second subtopic of Bar.', metadata={'Header 1': 'Foo', 'Header 2': 'Bar main section', 'Header 3': 'Bar subsection 2'}),
 Document(page_content='Baz', metadata={'Header 1': 'Foo'}),
 Document(page_content='Some text about Baz', metadata={'Header 1': 'Foo', 'Header 2': 'Baz'}),
 Document(page_content='Some concluding text about Foo', metadata={'Header 1': 'Foo'})]
```

要返回每个元素以及其关联的标题，请在实例化 `HTMLHeaderTextSplitter` 时指定 `return_each_element=True`：

```python
html_splitter = HTMLHeaderTextSplitter(
    headers_to_split_on,
    return_each_element=True,
)
html_header_splits_elements = html_splitter.split_text(html_string)
```

与上述情况相比，元素按其标题聚合：

```python
for element in html_header_splits[:2]:
    print(element)
```

```output
page_content='Foo'
page_content='Some intro text about Foo.  \nBar main section Bar subsection 1 Bar subsection 2' metadata={'Header 1': 'Foo'}
```

现在每个元素都作为一个独立的 `Document` 返回：

```python
for element in html_header_splits_elements[:3]:
    print(element)
```

```output
page_content='Foo'
page_content='Some intro text about Foo.' metadata={'Header 1': 'Foo'}
page_content='Bar main section Bar subsection 1 Bar subsection 2' metadata={'Header 1': 'Foo'}
```

### 2) 如何从 URL 或 HTML 文件进行分割：

要直接从 URL 读取，请将 URL 字符串传递给 `split_text_from_url` 方法。类似地，可以将本地 HTML 文件传递给 `split_text_from_file` 方法。

```python
url = "https://plato.stanford.edu/entries/goedel/"
headers_to_split_on = [
    ("h1", "Header 1"),
    ("h2", "Header 2"),
    ("h3", "Header 3"),
    ("h4", "Header 4"),
]
html_splitter = HTMLHeaderTextSplitter(headers_to_split_on)
# for local file use html_splitter.split_text_from_file(<path_to_file>)
html_header_splits = html_splitter.split_text_from_url(url)
```

### 2) 如何限制分块大小：

`HTMLHeaderTextSplitter` 是基于 HTML 标题进行分割的，可以与基于字符长度约束分割的另一个分割器（例如 `RecursiveCharacterTextSplitter`）组合使用。可以使用第二个分割器的 `.split_documents` 方法来实现。

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
chunk_size = 500
chunk_overlap = 30
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=chunk_size, chunk_overlap=chunk_overlap
)
# 分割
splits = text_splitter.split_documents(html_header_splits)
splits[80:85]
```

## 限制

从一个 HTML 文档到另一个 HTML 文档，结构变化可能会很大，而且 `HTMLHeaderTextSplitter` 会尝试将所有“相关”的标题附加到任何给定的块上，但有时可能会错过某些标题。例如，在以下新闻文章中（截至本文撰写时），文档的结构被设计成顶级标题的文本，虽然标记为“h1”，但实际上位于与我们期望的文本元素*不同*的子树中，因此我们可以观察到“h1”元素及其相关文本未出现在块元数据中（但在适用的情况下，我们确实看到“h2”及其相关文本）：

```python
url = "https://www.cnn.com/2023/09/25/weather/el-nino-winter-us-climate/index.html"
headers_to_split_on = [
    ("h1", "Header 1"),
    ("h2", "Header 2"),
]
html_splitter = HTMLHeaderTextSplitter(headers_to_split_on)
html_header_splits = html_splitter.split_text_from_url(url)
print(html_header_splits[1].page_content[:500])
```

```output
没有两个厄尔尼诺冬天是相同的，但许多冬天的温度和降水趋势是相似的。
厄尔尼诺冬天在美国大陆的平均条件。
一个主要原因是急流的位置，在厄尔尼诺冬天经常向南移动。根据美国国家海洋和大气管理局（NOAA）的说法，这种转变通常会给南部带来更多的雨水和更凉爽的天气，而北部则变得更干燥和更温暖。
由于急流本质上是一条空气之河，风暴会通过它流动，它们会受到急流位置的影响。这种移动会影响风暴的路径和强度，从而影响降水模式。这种变化可能会导致洪水、干旱和其他极端天气事件。
```