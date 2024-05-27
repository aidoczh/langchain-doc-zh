

# 非结构化文件

本文介绍如何使用 `Unstructured` 软件包加载多种类型的文件。`Unstructured` 目前支持加载文本文件、PowerPoint 文件、HTML、PDF、图像等。

```python
# # 安装软件包
%pip install --upgrade --quiet  "unstructured[all-docs]"
```

```python
# # 安装其他依赖项
# # https://github.com/Unstructured-IO/unstructured/blob/main/docs/source/installing.rst
# !brew install libmagic
# !brew install poppler
# !brew install tesseract
# # 如果解析 XML / HTML 文档：
# !brew install libxml2
# !brew install libxslt
```

```python
# import nltk
# nltk.download('punkt')
```

```python
from langchain_community.document_loaders import UnstructuredFileLoader
```

```python
loader = UnstructuredFileLoader("./example_data/state_of_the_union.txt")
```

```python
docs = loader.load()
```

```python
docs[0].page_content[:400]
```

```output
'女士们先生们，女副总统，我们的第一夫人和第二绅士。国会议员和内阁成员。最高法院大法官。我的美国同胞。\n\n去年，COVID-19让我们分开。今年，我们终于再次团聚。\n\n今晚，我们作为民主党人、共和党人和独立人士相聚。但更重要的是作为美国人。\n\n我们有责任对彼此，对美国人民，对宪法负责。'
```

### 加载文件列表

```python
files = ["./example_data/whatsapp_chat.txt", "./example_data/layout-parser-paper.pdf"]
```

```python
loader = UnstructuredFileLoader(files)
```

```python
docs = loader.load()
```

```python
docs[0].page_content[:400]
```

## 保留元素

在幕后，Unstructured 为不同的文本块创建不同的 "元素"。默认情况下，我们将它们组合在一起，但您可以通过指定 `mode="elements"` 轻松保持分离。

```python
loader = UnstructuredFileLoader(
    "./example_data/state_of_the_union.txt", mode="elements"
)
```

```python
docs = loader.load()
```

```python
docs[:5]
```

## 定义分区策略

Unstructured 文档加载器允许用户传入 `strategy` 参数，让 `unstructured` 知道如何对文档进行分区。当前支持的策略有 `"hi_res"`（默认）和 `"fast"`。高分辨率分区策略更准确，但处理时间更长。快速策略可以更快地对文档进行分区，但会牺牲准确性。并非所有文档类型都有单独的高分辨率和快速分区策略。对于这些文档类型，`strategy` 参数将被忽略。在某些情况下，如果缺少依赖项（即文档分区模型），高分辨率策略将退回到快速策略。以下是如何将策略应用于 `UnstructuredFileLoader`。

```python
from langchain_community.document_loaders import UnstructuredFileLoader
```

```python
loader = UnstructuredFileLoader(
    "layout-parser-paper-fast.pdf", strategy="fast", mode="elements"
)
```

```python
docs = loader.load()
```

```python
docs[:5]
```

## PDF 示例

处理 PDF 文档的方法与处理其他文档的方法完全相同。Unstructured 检测文件类型并提取相同类型的元素。操作模式有：

- `single` 所有元素的所有文本被合并为一个（默认）

- `elements` 保持各个元素的独立性

- `paged` 每页的文本只被合并

```python
!wget  https://raw.githubusercontent.com/Unstructured-IO/unstructured/main/example-docs/layout-parser-paper.pdf -P "../../"
```

```python
loader = UnstructuredFileLoader(
    "./example_data/layout-parser-paper.pdf", mode="elements"
)
```

```python
docs = loader.load()
```

```python
docs[:5]
```

```output
[Document(page_content='LayoutParser : A Uniﬁed Toolkit for Deep Learning Based Document Image Analysis', lookup_str='', metadata={'source': '../../layout-parser-paper.pdf'}, lookup_index=0),
 Document(page_content='Zejiang Shen 1 ( (ea)\n ), Ruochen Zhang 2 , Melissa Dell 3 , Benjamin Charles Germain Lee 4 , Jacob Carlson 3 , and Weining Li 5', lookup_str='', metadata={'source': '../../layout-parser-paper.pdf'}, lookup_index=0),
 Document(page_content='Allen Institute for AI shannons@allenai.org', lookup_str='', metadata={'source': '../../layout-parser-paper.pdf'}, lookup_index=0),
 Document(page_content='Brown University ruochen zhang@brown.edu', lookup_str='', metadata={'source': '../../layout-parser-paper.pdf'}, lookup_index=0),
 Document(page_content='Harvard University { melissadell,jacob carlson } @fas.harvard.edu', lookup_str='', metadata={'source': '../../layout-parser-paper.pdf'}, lookup_index=0)]
```

如果您需要在提取后对 `unstructured` 元素进行后处理，可以在实例化 `UnstructuredFileLoader` 时通过 `post_processors` 参数传入一个 `str` -> `str` 函数列表。这也适用于其他 Unstructured 加载器。以下是一个示例。

```python
from langchain_community.document_loaders import UnstructuredFileLoader
from unstructured.cleaners.core import clean_extra_whitespace
```

```python
loader = UnstructuredFileLoader(
    "./example_data/layout-parser-paper.pdf",
    mode="elements",
    post_processors=[clean_extra_whitespace],
)
```

```python
docs = loader.load()
```

```python
docs[:5]
```

```output
[Document(page_content='LayoutParser: A Uniﬁed Toolkit for Deep Learning Based Document Image Analysis', metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((157.62199999999999, 114.23496279999995), (157.62199999999999, 146.5141628), (457.7358962799999, 146.5141628), (457.7358962799999, 114.23496279999995)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'filename': 'layout-parser-paper.pdf', 'file_directory': './example_data', 'filetype': 'application/pdf', 'page_number': 1, 'category': 'Title'}),
 Document(page_content='Zejiang Shen1 ((cid:0)), Ruochen Zhang2, Melissa Dell3, Benjamin Charles Germain Lee4, Jacob Carlson3, and Weining Li5', metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((134.809, 168.64029940800003), (134.809, 192.2517444), (480.5464199080001, 192.2517444), (480.5464199080001, 168.64029940800003)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'filename': 'layout-parser-paper.pdf', 'file_directory': './example_data', 'filetype': 'application/pdf', 'page_number': 1, 'category': 'UncategorizedText'}),
 Document(page_content='1 Allen Institute for AI shannons@allenai.org 2 Brown University ruochen zhang@brown.edu 3 Harvard University {melissadell,jacob carlson}@fas.harvard.edu 4 University of Washington bcgl@cs.washington.edu 5 University of Waterloo w422li@uwaterloo.ca', metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((207.23000000000002, 202.57205439999996), (207.23000000000002, 311.8195408), (408.12676, 311.8195408), (408.12676, 202.57205439999996)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'filename': 'layout-parser-paper.pdf', 'file_directory': './example_data', 'filetype': 'application/pdf', 'page_number': 1, 'category': 'UncategorizedText'}),
 Document(page_content='1 2 0 2', metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((16.34, 213.36), (16.34, 253.36), (36.34, 253.36), (36.34, 213.36)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'filename': 'layout-parser-paper.pdf', 'file_directory': './example_data', 'filetype': 'application/pdf', 'page_number': 1, 'category': 'UncategorizedText'}),
```

## 非结构化 API

如果您想快速上手而又不想进行太多设置，只需运行 `pip install unstructured` 并使用 `UnstructuredAPIFileLoader` 或 `UnstructuredAPIFileIOLoader` 即可。这将使用托管的非结构化 API 处理您的文档。您可以在[这里](https://www.unstructured.io/api-key/)生成免费的非结构化 API 密钥。[非结构化文档](https://unstructured-io.github.io/unstructured/)页面将在可用时提供有关如何生成 API 密钥的说明。如果您想要自行托管非结构化 API 或在本地运行它，请查看[这里](https://github.com/Unstructured-IO/unstructured-api#dizzy-instructions-for-using-the-docker-image)的说明。

```python
from langchain_community.document_loaders import UnstructuredAPIFileLoader
```

```python
filenames = ["example_data/fake.docx", "example_data/fake-email.eml"]
```

```python
loader = UnstructuredAPIFileLoader(
    file_path=filenames[0],
    api_key="FAKE_API_KEY",
)
```

```python
docs = loader.load()
docs[0]
```

```output
Document(page_content='Lorem ipsum dolor sit amet.', metadata={'source': 'example_data/fake.docx'})
```

您还可以通过单个 API 批量处理多个文件，使用 `UnstructuredAPIFileLoader`。

```python
loader = UnstructuredAPIFileLoader(
    file_path=filenames,
    api_key="FAKE_API_KEY",
)
```

```python
docs = loader.load()
docs[0]
```

```output
Document(page_content='Lorem ipsum dolor sit amet.\n\nThis is a test email to use for unit tests.\n\nImportant points:\n\nRoses are red\n\nViolets are blue', metadata={'source': ['example_data/fake.docx', 'example_data/fake-email.eml']})
```