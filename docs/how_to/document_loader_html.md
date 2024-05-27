# 如何加载HTML

超文本标记语言（HTML）是用于在Web浏览器中显示的文档的标准标记语言。

这里介绍了如何将HTML文档加载到LangChain的[Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html#langchain_core.documents.base.Document)对象中，以便我们可以在下游使用。

解析HTML文件通常需要专门的工具。在这里，我们演示了如何通过[Unstructured](https://unstructured-io.github.io/unstructured/)和[BeautifulSoup4](https://beautiful-soup-4.readthedocs.io/en/latest/)进行解析，可以通过pip安装。请前往集成页面查找与其他服务的集成，例如[Azure AI Document Intelligence](/docs/integrations/document_loaders/azure_document_intelligence)或[FireCrawl](/docs/integrations/document_loaders/firecrawl)。

## 使用Unstructured加载HTML

```python
%pip install "unstructured[html]"
```

```python
from langchain_community.document_loaders import UnstructuredHTMLLoader
file_path = "../../../docs/integrations/document_loaders/example_data/fake-content.html"
loader = UnstructuredHTMLLoader(file_path)
data = loader.load()
print(data)
```

```output
[Document(page_content='My First Heading\n\nMy first paragraph.', metadata={'source': '../../../docs/integrations/document_loaders/example_data/fake-content.html'})]
```

## 使用BeautifulSoup4加载HTML

我们还可以使用BeautifulSoup4使用`BSHTMLLoader`加载HTML文档。这将将HTML中的文本提取到`page_content`中，并将页面标题提取到`metadata`的`title`中。

```python
%pip install bs4
```

```python
from langchain_community.document_loaders import BSHTMLLoader
loader = BSHTMLLoader(file_path)
data = loader.load()
print(data)
```

```output
[Document(page_content='\nTest Title\n\n\nMy First Heading\nMy first paragraph.\n\n\n', metadata={'source': '../../../docs/integrations/document_loaders/example_data/fake-content.html', 'title': 'Test Title'})]
```