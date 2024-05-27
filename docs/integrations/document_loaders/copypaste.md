# 复制粘贴

本文介绍如何从你想要复制粘贴的内容中加载一个文档对象。在这种情况下，你甚至不需要使用 DocumentLoader，而是可以直接构造 Document 对象。

```python
from langchain_core.documents import Document
```

```python
text = ".....将你复制粘贴的文本放在这里......"
```

```python
doc = Document(page_content=text)
```

## 元数据

如果你想添加关于这段文本的来源的元数据，你可以很容易地使用 metadata 键。

```python
metadata = {"source": "internet", "date": "Friday"}
```

```python
doc = Document(page_content=text, metadata=metadata)
```