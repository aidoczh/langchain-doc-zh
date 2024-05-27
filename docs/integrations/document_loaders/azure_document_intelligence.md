# Azure AI 文档智能

[**Azure AI 文档智能**](https://aka.ms/doc-intelligence)（以前称为 `Azure Form Recognizer`）是基于机器学习的服务，可以从数字或扫描的 PDF、图像、Office 和 HTML 文件中提取文本（包括手写文字）、表格、文档结构（例如标题、章节标题等）和键值对。

文档智能支持 `PDF`、`JPEG/JPG`、`PNG`、`BMP`、`TIFF`、`HEIF`、`DOCX`、`XLSX`、`PPTX` 和 `HTML` 格式的文件。

当前的 `Document Intelligence` 加载器实现可以将内容逐页整合，并将其转换为 LangChain 文档。默认输出格式为 markdown，可以轻松地与 `MarkdownHeaderTextSplitter` 进行语义文档分块。您还可以使用 `mode="single"` 或 `mode="page"` 来返回单页纯文本或按页拆分的文档。

## 先决条件

在以下 3 个预览区域之一拥有 Azure AI 文档智能资源：**东部美国**、**西部美国2**、**西欧** - 如果没有，请参阅[此文档](https://learn.microsoft.com/azure/ai-services/document-intelligence/create-document-intelligence-resource?view=doc-intel-4.0.0)进行创建。您将需要将 `<endpoint>` 和 `<key>` 作为参数传递给加载器。

```python
%pip install --upgrade --quiet  langchain langchain-community azure-ai-documentintelligence
```

## 示例 1

第一个示例使用本地文件，该文件将被发送到 Azure AI 文档智能。

通过初始化文档分析客户端，我们可以继续创建 DocumentIntelligenceLoader 的实例：

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

默认输出包含一个 LangChain 文档，其内容格式为 markdown：

```python
documents
```

## 示例 2

输入文件也可以是公共 URL 路径。例如，https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/rest-api/layout.png。

```python
url_path = "<url>"
loader = AzureAIDocumentIntelligenceLoader(
    api_endpoint=endpoint, api_key=key, url_path=url_path, api_model="prebuilt-layout"
)
documents = loader.load()
```

```python
documents
```

## 示例 3

您还可以指定 `mode="page"` 以按页加载文档。

```python
from langchain_community.document_loaders import AzureAIDocumentIntelligenceLoader
file_path = "<filepath>"
endpoint = "<endpoint>"
key = "<key>"
loader = AzureAIDocumentIntelligenceLoader(
    api_endpoint=endpoint,
    api_key=key,
    file_path=file_path,
    api_model="prebuilt-layout",
    mode="page",
)
documents = loader.load()
```

输出将作为列表中的单独文档存储每一页的内容：

```python
for document in documents:
    print(f"Page Content: {document.page_content}")
    print(f"Metadata: {document.metadata}")
```

## 示例 4

您还可以指定 `analysis_feature=["ocrHighResolution"]` 以启用附加功能。更多信息，请参阅：https://aka.ms/azsdk/python/documentintelligence/analysisfeature。

```python
from langchain_community.document_loaders import AzureAIDocumentIntelligenceLoader
file_path = "<filepath>"
endpoint = "<endpoint>"
key = "<key>"
analysis_features = ["ocrHighResolution"]
loader = AzureAIDocumentIntelligenceLoader(
    api_endpoint=endpoint,
    api_key=key,
    file_path=file_path,
    api_model="prebuilt-layout",
    analysis_features=analysis_features,
)
documents = loader.load()
```

输出包含具有高分辨率附加功能的 LangChain 文档：

```python
documents
```