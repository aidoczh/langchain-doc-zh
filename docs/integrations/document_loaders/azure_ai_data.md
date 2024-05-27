# Azure AI 数据

[Azure AI Studio](https://ai.azure.com/) 提供了将数据资产上传到云存储并从以下来源注册现有数据资产的功能：

- `Microsoft OneLake`

- `Azure Blob Storage`

- `Azure Data Lake gen 2`

这种方法相对于 `AzureBlobStorageContainerLoader` 和 `AzureBlobStorageFileLoader` 的优势在于，身份验证可以无缝地处理到云存储。您可以使用基于身份的数据访问控制或基于凭据的方式（例如 SAS 令牌、帐户密钥）。在基于凭据的数据访问方式中，您无需在代码中指定密钥或设置密钥保管库 - 系统会为您处理。

本笔记本介绍了如何从 AI Studio 中的数据资产加载文档对象。

```python
%pip install --upgrade --quiet  azureml-fsspec, azure-ai-generative
```

```python
from azure.ai.resources.client import AIClient
from azure.identity import DefaultAzureCredential
from langchain_community.document_loaders import AzureAIDataLoader
```

```python
# 创建到您的项目的连接
client = AIClient(
    credential=DefaultAzureCredential(),
    subscription_id="<subscription_id>",
    resource_group_name="<resource_group_name>",
    project_name="<project_name>",
)
```

```python
# 获取数据资产的最新版本
data_asset = client.data.get(name="<data_asset_name>", label="latest")
```

```python
# 加载数据资产
loader = AzureAIDataLoader(url=data_asset.path)
```

```python
loader.load()
```

```output
[Document(page_content='Lorem ipsum dolor sit amet.', lookup_str='', metadata={'source': '/var/folders/y6/8_bzdg295ld6s1_97_12m4lr0000gn/T/tmpaa9xl6ch/fake.docx'}, lookup_index=0)]
```

## 指定 glob 模式

您还可以指定 glob 模式，以更精细地控制要加载的文件。在下面的示例中，只会加载具有 `pdf` 扩展名的文件。

```python
loader = AzureAIDataLoader(url=data_asset.path, glob="*.pdf")
```

```python
loader.load()
```

```output
[Document(page_content='Lorem ipsum dolor sit amet.', lookup_str='', metadata={'source': '/var/folders/y6/8_bzdg295ld6s1_97_12m4lr0000gn/T/tmpujbkzf_l/fake.docx'}, lookup_index=0)]
```