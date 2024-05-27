# 微软 Excel

`UnstructuredExcelLoader` 用于加载 `Microsoft Excel` 文件。该加载器适用于 `.xlsx` 和 `.xls` 文件。页面内容将是 Excel 文件的原始文本。如果您在 "elements" 模式下使用加载器，则可以在文档元数据中的 `text_as_html` 键下找到 Excel 文件的 HTML 表示形式。

```python
from langchain_community.document_loaders import UnstructuredExcelLoader
```

```python
loader = UnstructuredExcelLoader("example_data/stanley-cups.xlsx", mode="elements")
docs = loader.load()
docs[0]
```

```output
Document(page_content='\n  \n    \n      Team\n      Location\n      Stanley Cups\n    \n    \n      Blues\n      STL\n      1\n    \n    \n      Flyers\n      PHI\n      2\n    \n    \n      Maple Leafs\n      TOR\n      13\n    \n  \n', metadata={'source': 'example_data/stanley-cups.xlsx', 'filename': 'stanley-cups.xlsx', 'file_directory': 'example_data', 'filetype': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet', 'page_number': 1, 'page_name': 'Stanley Cups', 'text_as_html': '<table border="1" class="dataframe">\n  <tbody>\n    <tr>\n      <td>Team</td>\n      <td>Location</td>\n      <td>Stanley Cups</td>\n    </tr>\n    <tr>\n      <td>Blues</td>\n      <td>STL</td>\n      <td>1</td>\n    </tr>\n    <tr>\n      <td>Flyers</td>\n      <td>PHI</td>\n      <td>2</td>\n    </tr>\n    <tr>\n      <td>Maple Leafs</td>\n      <td>TOR</td>\n      <td>13</td>\n    </tr>\n  </tbody>\n</table>', 'category': 'Table'})
```

## 使用 Azure AI 文档智能

>[Azure AI 文档智能](https://aka.ms/doc-intelligence)（以前称为 `Azure Form Recognizer`）是基于机器学习的服务，可从数字或扫描的 PDF、图像、Office 和 HTML 文件中提取文本（包括手写）、表格、文档结构（例如标题、章节标题等）和键值对。

>

>文档智能支持 `PDF`、`JPEG/JPG`、`PNG`、`BMP`、`TIFF`、`HEIF`、`DOCX`、`XLSX`、`PPTX` 和 `HTML`。

使用 `Document Intelligence` 的当前实现可以将内容逐页合并，并将其转换为 LangChain 文档。默认的输出格式是 markdown，可以轻松地与 `MarkdownHeaderTextSplitter` 链接以进行语义文档分块。您还可以使用 `mode="single"` 或 `mode="page"` 返回单个页面中的纯文本或按页面拆分的文档。

### 先决条件

在以下 3 个预览区域之一拥有 Azure AI 文档智能资源：**东部美国**、**西部美国2**、**西欧** - 如果没有，请按照[此文档](https://learn.microsoft.com/azure/ai-services/document-intelligence/create-document-intelligence-resource?view=doc-intel-4.0.0)创建一个。您将会将 `<endpoint>` 和 `<key>` 作为参数传递给加载器。

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