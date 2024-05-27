# CoNLL-U

>[CoNLL-U](https://universaldependencies.org/format.html) 是 CoNLL-X 格式的修订版本。注释以纯文本文件的形式进行编码（UTF-8 编码，使用 LF 字符作为换行符，文件末尾包含一个 LF 字符），文件中包含三种类型的行：

>- 单词行，包含由单个制表符分隔的 10 个字段的单词/标记的注释；请参见下文。

>- 空行，标记句子边界。

>- 以井号（#）开头的注释行。

这是一个加载 [CoNLL-U](https://universaldependencies.org/format.html) 格式文件的示例。整个文件被视为一个文档。示例数据（`conllu.conllu`）基于标准的 UD/CoNLL-U 示例之一。

```python
from langchain_community.document_loaders import CoNLLULoader
```

```python
loader = CoNLLULoader("example_data/conllu.conllu")
```

```python
document = loader.load()
```

```python
document
```

```output
[Document(page_content='They buy and sell books.', metadata={'source': 'example_data/conllu.conllu'})]
```