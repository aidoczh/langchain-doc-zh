# 电子邮件

本笔记展示了如何加载电子邮件（`.eml`）或 `Microsoft Outlook`（`.msg`）文件。

## 使用 Unstructured

```python
%pip install --upgrade --quiet  unstructured
```

```python
from langchain_community.document_loaders import UnstructuredEmailLoader
```

```python
loader = UnstructuredEmailLoader("example_data/fake-email.eml")
```

```python
data = loader.load()
```

```python
data
```

```output
[Document(page_content='这是一个用于单元测试的测试电子邮件。\n\n重要内容：\n\n玫瑰是红色的\n\n紫罗兰是蓝色的', metadata={'source': 'example_data/fake-email.eml'})]
```

### 保留元素

在幕后，Unstructured 为不同的文本块创建不同的“元素”。默认情况下，我们将它们合并在一起，但您可以通过指定 `mode="elements"` 轻松保持分离。

```python
loader = UnstructuredEmailLoader("example_data/fake-email.eml", mode="elements")
```

```python
data = loader.load()
```

```python
data[0]
```

```output
Document(page_content='这是一个用于单元测试的测试电子邮件。', metadata={'source': 'example_data/fake-email.eml', 'filename': 'fake-email.eml', 'file_directory': 'example_data', 'date': '2022-12-16T17:04:16-05:00', 'filetype': 'message/rfc822', 'sent_from': ['Matthew Robinson <mrobinson@unstructured.io>'], 'sent_to': ['Matthew Robinson <mrobinson@unstructured.io>'], 'subject': 'Test Email', 'category': 'NarrativeText'})
```

### 处理附件

您可以通过在构造函数中设置 `process_attachments=True` 来使用 `UnstructuredEmailLoader` 处理附件。默认情况下，附件将使用 `unstructured` 中的 `partition` 函数进行分区。您可以通过将函数传递给 `attachment_partitioner` 关键字参数来使用不同的分区函数。

```python
loader = UnstructuredEmailLoader(
    "example_data/fake-email.eml",
    mode="elements",
    process_attachments=True,
)
```

```python
data = loader.load()
```

```python
data[0]
```

```output
Document(page_content='这是一个用于单元测试的测试电子邮件。', metadata={'source': 'example_data/fake-email.eml', 'filename': 'fake-email.eml', 'file_directory': 'example_data', 'date': '2022-12-16T17:04:16-05:00', 'filetype': 'message/rfc822', 'sent_from': ['Matthew Robinson <mrobinson@unstructured.io>'], 'sent_to': ['Matthew Robinson <mrobinson@unstructured.io>'], 'subject': 'Test Email', 'category': 'NarrativeText'})
```

## 使用 OutlookMessageLoader

```python
%pip install --upgrade --quiet  extract_msg
```

```python
from langchain_community.document_loaders import OutlookMessageLoader
```

```python
loader = OutlookMessageLoader("example_data/fake-email.msg")
```

```python
data = loader.load()
```

```python
data[0]
```

```output
Document(page_content='这是一个用于测试 MS Outlook MSG Extractor 的测试电子邮件\r\n\r\n\r\n-- \r\n\r\n\r\n祝好\r\n\r\n\r\n\r\n\r\nBrian Zhou\r\n\r\n', metadata={'subject': 'Test for TIF files', 'sender': 'Brian Zhou <brizhou@gmail.com>', 'date': 'Mon, 18 Nov 2013 16:26:24 +0800'})
```