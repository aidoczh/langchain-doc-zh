# Iugu

[Iugu](https://www.iugu.com/) 是一家巴西服务和软件即服务（SaaS）公司。它提供用于电子商务网站和移动应用程序的支付处理软件和应用程序编程接口。

本文介绍了如何从 `Iugu REST API` 加载数据，以便将其转换为可被 LangChain 吸收的格式，并提供了向量化的示例用法。

```python
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.document_loaders import IuguLoader
```

Iugu API 需要访问令牌，该访问令牌可以在 Iugu 仪表板中找到。

此文档加载程序还需要一个 `resource` 选项，用于定义要加载的数据。

以下资源可用：

`Documentation` [Documentation](https://dev.iugu.com/reference/metadados)

```python
iugu_loader = IuguLoader("charges")
```

```python
# 从加载程序创建一个 vectorstore retriever
# 有关更多详细信息，请参阅 https://python.langchain.com/en/latest/modules/data_connection/getting_started.html
index = VectorstoreIndexCreator().from_loaders([iugu_loader])
iugu_doc_retriever = index.vectorstore.as_retriever()
```