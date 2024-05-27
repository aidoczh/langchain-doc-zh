# Vsdx

> 一个 [visio 文件](https://fr.wikipedia.org/wiki/Microsoft_Visio)（扩展名为 .vsdx）与 Microsoft Visio 软件相关联，这是一款用于创建图表的软件。它存储有关图表结构、布局和图形元素的信息。这种格式有助于在业务、工程和计算机科学等领域创建和共享可视化内容。

Visio 文件可以包含多个页面。其中一些页面可能用作其他页面的背景，这可能跨越多个层。这个**加载器**从每个页面及其关联页面中提取文本内容，使得可以提取每个页面的所有可见文本，类似于 OCR 算法的功能。

**警告**：只有扩展名为 **.vsdx** 的 Visio 文件与此加载器兼容。扩展名为 .vsd 的文件等不兼容，因为它们无法转换为压缩的 XML。

```python
from langchain_community.document_loaders import VsdxLoader
```

```python
loader = VsdxLoader(file_path="./example_data/fake.vsdx")
documents = loader.load()
```

**显示加载的文档**

```python
for i, doc in enumerate(documents):
    print(f"\n------ 第 {doc.metadata['page']} 页 ------")
    print(f"标题页：{doc.metadata['page_name']}")
    print(f"来源：{doc.metadata['source']}")
    print("\n==> 内容 <== ")
    print(doc.page_content)
```

```output
------ 第 0 页 ------
标题页：摘要
来源：./example_data/fake.vsdx
==> 内容 <== 
创建者
创建日期
修改者
修改日期
版本
标题
Florian MOREL
2024-01-14
FLORIAN Morel
今天
0.0.0.0.0.1
这是一个标题
世界上最好的标题
这是一个箭头
这是地球
这是一个有界箭头
...
```