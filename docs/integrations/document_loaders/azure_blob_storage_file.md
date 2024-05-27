# Azure Blob 存储文件

[**Azure Files**](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-introduction) 提供了在云中可通过行业标准的 Server Message Block (`SMB`) 协议、Network File System (`NFS`) 协议和 `Azure Files REST API` 访问的完全托管文件共享。

这里介绍了如何从 Azure Files 中加载文档对象。

```python
%pip install --upgrade --quiet  azure-storage-blob
```

```python
from langchain_community.document_loaders import AzureBlobStorageFileLoader
```

```python
loader = AzureBlobStorageFileLoader(
    conn_str="<connection string>",
    container="<container name>",
    blob_name="<blob name>",
)
```

```python
loader.load()
```

```output
[Document(page_content='Lorem ipsum dolor sit amet.', lookup_str='', metadata={'source': '/var/folders/y6/8_bzdg295ld6s1_97_12m4lr0000gn/T/tmpxvave6wl/fake.docx'}, lookup_index=0)]
```

![Azure Blob Storage](https://example.com/azure_blob_storage.png)