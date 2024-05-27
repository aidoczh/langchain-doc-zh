# Microsoft Word

>[Microsoft Word](https://www.microsoft.com/zh-cn/microsoft-365/word) 是由 Microsoft 开发的文字处理软件。

本文介绍了如何将 `Word` 文档加载为我们可以在下游使用的文档格式。

## 使用 Docx2txt

使用 `Docx2txt` 将 .docx 文件加载为文档。

```python
%pip install --upgrade --quiet  docx2txt
```

```python
from langchain_community.document_loaders import Docx2txtLoader
```

```python
loader = Docx2txtLoader("example_data/fake.docx")
```

```python
data = loader.load()
```

```python
data
```

```output
[Document(page_content='Lorem ipsum dolor sit amet.', metadata={'source': 'example_data/fake.docx'})]
```

## 使用 Unstructured

```python
from langchain_community.document_loaders import UnstructuredWordDocumentLoader
```

```python
loader = UnstructuredWordDocumentLoader("example_data/fake.docx")
```

```python
data = loader.load()
```

```python
data
```

```output
[Document(page_content='Lorem ipsum dolor sit amet.', lookup_str='', metadata={'source': 'fake.docx'}, lookup_index=0)]
```

### 保留元素

在底层，Unstructured 为不同的文本块创建不同的 "元素"。默认情况下，我们将它们合并在一起，但您可以通过指定 `mode="elements"` 来保留该分离。

```python
loader = UnstructuredWordDocumentLoader("example_data/fake.docx", mode="elements")
```

```python
data = loader.load()
```

```python
data[0]
```

```output
Document(page_content='Lorem ipsum dolor sit amet.', lookup_str='', metadata={'source': 'fake.docx', 'filename': 'fake.docx', 'category': 'Title'}, lookup_index=0)
```

## 使用 Azure AI 文档智能

>[Azure AI 文档智能](https://aka.ms/doc-intelligence)（以前称为 `Azure 表单识别器`）是基于机器学习的服务，可从数字或扫描的 PDF、图像、Office 和 HTML 文件中提取文本（包括手写）、表格、文档结构（例如标题、章节标题等）和键值对。

文档智能支持 `PDF`、`JPEG/JPG`、`PNG`、`BMP`、`TIFF`、`HEIF`、`DOCX`、`XLSX`、`PPTX` 和 `HTML`。

使用 `Document Intelligence` 的当前实现可以将内容逐页合并，并将其转换为 LangChain 文档。默认输出格式为 markdown，可以与 `MarkdownHeaderTextSplitter` 轻松链接以进行语义文档分块。您还可以使用 `mode="single"` 或 `mode="page"` 返回单个页面或按页面拆分的纯文本。

## 先决条件

在以下 3 个预览区域之一拥有 Azure AI 文档智能资源：**East US**、**West US2**、**West Europe** - 如果没有，请按照[此文档](https://learn.microsoft.com/azure/ai-services/document-intelligence/create-document-intelligence-resource?view=doc-intel-4.0.0)创建一个。您将把 `<endpoint>` 和 `<key>` 作为参数传递给加载器。

%pip install --upgrade --quiet  langchain langchain-community azure-ai-documentintelligence

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