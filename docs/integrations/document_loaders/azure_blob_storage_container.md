# Azure Blob Storage 容器

[**Azure Blob Storage**](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) 是微软的云对象存储解决方案。Blob Storage 专为存储大量非结构化数据进行了优化。非结构化数据是指不符合特定数据模型或定义的数据，例如文本或二进制数据。

`Azure Blob Storage` 的设计用途包括：

- 直接向浏览器提供图像或文档。

- 存储用于分布式访问的文件。

- 流式传输视频和音频。

- 写入日志文件。

- 存储用于备份和恢复、灾难恢复和归档的数据。

- 存储供本地或 Azure 托管服务进行分析的数据。

本文介绍了如何从 `Azure Blob Storage` 容器中加载文档对象。

```python
%pip install --upgrade --quiet  azure-storage-blob
```

```python
from langchain_community.document_loaders import AzureBlobStorageContainerLoader
```

```python
loader = AzureBlobStorageContainerLoader(conn_str="<conn_str>", container="<container>")
```

```python
loader.load()
```

```output
[Document(page_content='Lorem ipsum dolor sit amet.', lookup_str='', metadata={'source': '/var/folders/y6/8_bzdg295ld6s1_97_12m4lr0000gn/T/tmpaa9xl6ch/fake.docx'}, lookup_index=0)]
```

## 指定前缀

您还可以指定前缀，以更精细地控制要加载的文件。

```python
loader = AzureBlobStorageContainerLoader(
    conn_str="<conn_str>", container="<container>", prefix="<prefix>"
)
```

```python
loader.load()
```

```output
[Document(page_content='Lorem ipsum dolor sit amet.', lookup_str='', metadata={'source': '/var/folders/y6/8_bzdg295ld6s1_97_12m4lr0000gn/T/tmpujbkzf_l/fake.docx'}, lookup_index=0)]
```