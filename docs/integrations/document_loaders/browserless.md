# Browserless

Browserless 是一个允许你在云端运行无头 Chrome 实例的服务。这是一个在大规模运行基于浏览器的自动化任务时，无需担心管理自己的基础设施的绝佳方式。

要将 Browserless 用作文档加载器，可以像在这个笔记本中所示那样初始化一个 `BrowserlessLoader` 实例。请注意，默认情况下，`BrowserlessLoader` 返回页面 `body` 元素的 `innerText`。要禁用这一设置并获取原始 HTML，请将 `text_content` 设置为 `False`。

```python
from langchain_community.document_loaders import BrowserlessLoader
```

```python
BROWSERLESS_API_TOKEN = "YOUR_BROWSERLESS_API_TOKEN"
```

```python
loader = BrowserlessLoader(
    api_token=BROWSERLESS_API_TOKEN,
    urls=[
        "https://en.wikipedia.org/wiki/Document_classification",
    ],
    text_content=True,
)
documents = loader.load()
print(documents[0].page_content[:1000])
```

```output
Jump to content
Main menu
Search
Create account
Log in
Personal tools
Toggle the table of contents
Document classification
17 languages
Article
Talk
Read
Edit
View history
Tools
From Wikipedia, the free encyclopedia
Document classification or document categorization is a problem in library science, information science and computer science. The task is to assign a document to one or more classes or categories. This may be done "manually" (or "intellectually") or algorithmically. The intellectual classification of documents has mostly been the province of library science, while the algorithmic classification of documents is mainly in information science and computer science. The problems are overlapping, however, and there is therefore interdisciplinary research on document classification.
The documents to be classified may be texts, images, music, etc. Each kind of document possesses its special classification problems. When not otherwise specified, text classification is implied.
Do
```