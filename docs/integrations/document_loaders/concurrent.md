# 并发加载器

对于那些选择优化工作流程的人来说，与通用加载器的工作方式相同，但具有并发加载功能。

```python
from langchain_community.document_loaders import ConcurrentLoader
```

```python
loader = ConcurrentLoader.from_filesystem("example_data/", glob="**/*.txt")
```

```python
files = loader.load()
```

```python
len(files)
```

```output
2
```