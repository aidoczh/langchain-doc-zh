# Grobid

GROBID 是一个用于提取、解析和重构原始文档的机器学习库。

它的设计初衷是用于解析学术论文，特别是在这方面表现得非常出色。需要注意的是，如果提供给 GROBID 的文章是大型文档（例如论文），超过一定数量的元素，可能无法处理。

---

最佳的安装方法是通过 Docker 安装 GROBID，详情请参见 [https://grobid.readthedocs.io/en/latest/Grobid-docker/](https://grobid.readthedocs.io/en/latest/Grobid-docker/)。

（注意：其他指令可以在[这里](/docs/integrations/providers/grobid)找到。）

一旦 GROBID 安装并运行起来，您可以按照下面的描述进行交互。

现在，我们可以使用数据加载器。

```python
from langchain_community.document_loaders.generic import GenericLoader
from langchain_community.document_loaders.parsers import GrobidParser
```

```python
loader = GenericLoader.from_filesystem(
    "../Papers/",
    glob="*",
    suffixes=[".pdf"],
    parser=GrobidParser(segment_sentences=False),
)
docs = loader.load()
```

```python
docs[3].page_content
```

```output
'与 Chinchilla、PaLM 或 GPT-3 不同，我们只使用公开可用的数据，使我们的工作与开源兼容，而大多数现有模型依赖的数据要么不公开可用，要么没有记录（例如“Books -2TB”或“社交媒体对话”）。也存在一些例外，特别是 OPT（Zhang 等，2022）、GPT-NeoX（Black 等，2022）、BLOOM（Scao 等，2022）和 GLM（Zeng 等，2022），但没有一个能与 PaLM-62B 或 Chinchilla 竞争。'
```

```python
docs[3].metadata
```

```output
{'text': '与 Chinchilla、PaLM 或 GPT-3 不同，我们只使用公开可用的数据，使我们的工作与开源兼容，而大多数现有模型依赖的数据要么不公开可用，要么没有记录（例如“Books -2TB”或“社交媒体对话”）。也存在一些例外，特别是 OPT（Zhang 等，2022）、GPT-NeoX（Black 等，2022）、BLOOM（Scao 等，2022）和 GLM（Zeng 等，2022），但没有一个能与 PaLM-62B 或 Chinchilla 竞争。',
 'para': '2',
 'bboxes': "[[{'page': '1', 'x': '317.05', 'y': '509.17', 'h': '207.73', 'w': '9.46'}, {'page': '1', 'x': '306.14', 'y': '522.72', 'h': '220.08', 'w': '9.46'}, {'page': '1', 'x': '306.14', 'y': '536.27', 'h': '218.27', 'w': '9.46'}, {'page': '1', 'x': '306.14', 'y': '549.82', 'h': '218.65', 'w': '9.46'}, {'page': '1', 'x': '306.14', 'y': '563.37', 'h': '136.98', 'w': '9.46'}], [{'page': '1', 'x': '446.49', 'y': '563.37', 'h': '78.11', 'w': '9.46'}, {'page': '1', 'x': '304.69', 'y': '576.92', 'h': '138.32', 'w': '9.46'}], [{'page': '1', 'x': '447.75', 'y': '576.92', 'h': '76.66', 'w': '9.46'}, {'page': '1', 'x': '306.14', 'y': '590.47', 'h': '219.63', 'w': '9.46'}, {'page': '1', 'x': '306.14', 'y': '604.02', 'h': '218.27', 'w': '9.46'}, {'page': '1', 'x': '306.14', 'y': '617.56', 'h': '218.27', 'w': '9.46'}, {'page': '1', 'x': '306.14', 'y': '631.11', 'h': '220.18', 'w': '9.46'}]]",
 'pages': "('1', '1')",
 'section_title': 'Introduction',
 'section_number': '1',
 'paper_title': 'LLaMA: Open and Efficient Foundation Language Models',
 'file_path': '/Users/31treehaus/Desktop/Papers/2302.13971.pdf'}
```