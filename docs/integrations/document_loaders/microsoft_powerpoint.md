# 微软 PowerPoint

[微软 PowerPoint](https://en.wikipedia.org/wiki/Microsoft_PowerPoint) 是微软的一款演示文稿程序。

这里介绍了如何将 `Microsoft PowerPoint` 文档加载到我们可以在下游使用的文档格式中。

```python
from langchain_community.document_loaders import UnstructuredPowerPointLoader
```

```python
loader = UnstructuredPowerPointLoader("example_data/fake-power-point.pptx")
```

```python
data = loader.load()
```

```python
data
```

```output
[Document(page_content='添加项目符号幻灯片\n\n查找项目符号幻灯片布局\n\n使用 _TextFrame.text 添加第一个项目符号\n\n使用 _TextFrame.add_paragraph() 添加后续项目符号\n\n这里有很多文本！\n\n这是文本框中的一些文本！', metadata={'source': 'example_data/fake-power-point.pptx'})]
```

### 保留元素

在幕后，`Unstructured` 为不同的文本块创建不同的 "元素"。默认情况下，我们将它们组合在一起，但您可以通过指定 `mode="elements"` 轻松保持分离。

```python
loader = UnstructuredPowerPointLoader(
    "example_data/fake-power-point.pptx", mode="elements"
)
```

```python
data = loader.load()
```

```python
data[0]
```

```output
Document(page_content='添加项目符号幻灯片', lookup_str='', metadata={'source': 'example_data/fake-power-point.pptx'}, lookup_index=0)
```

## 使用 Azure AI 文档智能

[Azure AI 文档智能](https://aka.ms/doc-intelligence)（以前称为 `Azure Form Recognizer`）是一种基于机器学习的服务，可以从数字或扫描的 PDF、图像、Office 和 HTML 文件中提取文本（包括手写文本）、表格、文档结构（例如标题、章节标题等）和键值对。

文档智能支持 `PDF`、`JPEG/JPG`、`PNG`、`BMP`、`TIFF`、`HEIF`、`DOCX`、`XLSX`、`PPTX` 和 `HTML`。

使用 `Document Intelligence` 的当前实现可以将内容逐页合并，并将其转换为 LangChain 文档。默认输出格式为 markdown，可以轻松地与 `MarkdownHeaderTextSplitter` 链接以进行语义文档分块。您还可以使用 `mode="single"` 或 `mode="page"` 返回单页或按页拆分的纯文本文档。

## 先决条件

在三个预览区域之一拥有 Azure AI 文档智能资源：**东部美国**、**西部美国2**、**西欧** - 如果没有，请按照[此文档](https://learn.microsoft.com/azure/ai-services/document-intelligence/create-document-intelligence-resource?view=doc-intel-4.0.0)创建一个。您将会将 `<endpoint>` 和 `<key>` 作为参数传递给加载器。

```python
%pip install --upgrade --quiet  langchain langchain-community azure-ai-documentintelligence
```

```python
from langchain_community.document_loaders import AzureAIDocumentIntelligenceLoader
file_path = "<filepath>"
endpoint = "<endpoint>"
key = "<key>"
loader = AzureAIDocumentIntelligenceLoader(
    api_endpoint=endpoint, api_key=key, file_path=file_path, api_model="prebuilt-layout"
)
documents = loader.load()
```