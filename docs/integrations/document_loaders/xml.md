# XML

`UnstructuredXMLLoader` 用于加载 `XML` 文件。该加载器适用于 `.xml` 文件。页面内容将是从 XML 标签中提取的文本。

```python
from langchain_community.document_loaders import UnstructuredXMLLoader
```

```python
loader = UnstructuredXMLLoader(
    "example_data/factbook.xml",
)
docs = loader.load()
docs[0]
```

```output
Document(page_content='美国\n\n华盛顿特区\n\n乔·拜登\n\n棒球\n\n加拿大\n\n渥太华\n\n贾斯汀·特鲁多\n\n曲棍球\n\n法国\n\n巴黎\n\n埃马纽埃尔·马克龙\n\n足球\n\n特立尼达和多巴哥\n\n西班牙港\n\n基思·罗利\n\n田径', metadata={'source': 'example_data/factbook.xml'})
```