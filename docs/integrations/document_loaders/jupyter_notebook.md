# Jupyter Notebook

[Jupyter Notebook](https://en.wikipedia.org/wiki/Project_Jupyter#Applications)（前身为 `IPython Notebook`）是一个基于网络的交互式计算环境，用于创建笔记本文档。

本笔记本介绍如何将 `Jupyter notebook (.html)` 中的数据加载到适合 LangChain 的格式中。

```python
from langchain_community.document_loaders import NotebookLoader
```

```python
loader = NotebookLoader(
    "example_data/notebook.html",
    include_outputs=True,
    max_output_length=20,
    remove_newline=True,
)
```

`NotebookLoader.load()` 将 `.html` 笔记本文件加载到一个 `Document` 对象中。

**参数**：

- `include_outputs`（布尔值）：是否在结果文档中包含单元格输出（默认为 False）。

- `max_output_length`（整数）：要包含在每个单元格输出中的最大字符数（默认为 10）。

- `remove_newline`（布尔值）：是否从单元格源和输出中删除换行符（默认为 False）。

- `traceback`（布尔值）：是否包含完整的回溯信息（默认为 False）。

```python
loader.load()
```

```output
[Document(page_content='\'markdown\' cell: \'[\'# Notebook\', \'\', \'This notebook covers how to load data from an .html notebook into a format suitable by LangChain.\']\'\n\n \'code\' cell: \'[\'from langchain_community.document_loaders import NotebookLoader\']\'\n\n \'code\' cell: \'[\'loader = NotebookLoader("example_data/notebook.html")\']\'\n\n \'markdown\' cell: \'[\'`NotebookLoader.load()` loads the `.html` notebook file into a `Document` object.\', \'\', \'**Parameters**:\', \'\', \'* `include_outputs` (bool): whether to include cell outputs in the resulting document (default is False).\', \'* `max_output_length` (int): the maximum number of characters to include from each cell output (default is 10).\', \'* `remove_newline` (bool): whether to remove newline characters from the cell sources and outputs (default is False).\', \'* `traceback` (bool): whether to include full traceback (default is False).\']\'\n\n \'code\' cell: \'[\'loader.load(include_outputs=True, max_output_length=20, remove_newline=True)\']\'\n\n', metadata={'source': 'example_data/notebook.html'})]
```