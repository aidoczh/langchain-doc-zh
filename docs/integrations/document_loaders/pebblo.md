# Pebblo 安全文档加载器

[Pebblo](https://daxa-ai.github.io/pebblo/) 可以帮助开发人员在不担心组织的合规性和安全要求的情况下加载数据并推广他们的 Gen AI 应用程序。该项目可以识别加载数据中的语义主题和实体，并在用户界面或 PDF 报告中对其进行总结。

Pebblo 有两个组件。

1. Pebblo 安全文档加载器（用于 Langchain）

1. Pebblo 服务器

本文档描述了如何使用 Pebblo 安全文档加载器来增强现有的 Langchain 文档加载器，以便深入了解加载到 Gen-AI Langchain 应用程序中的主题和实体类型。有关 `Pebblo 服务器` 的详细信息，请参阅此 [pebblo server](https://daxa-ai.github.io/pebblo/daemon) 文档。

Pebblo 安全加载器可以为 Langchain 的 `DocumentLoader` 提供安全的数据摄取。这是通过将文档加载器调用包装在 `Pebblo 安全文档加载器` 中来实现的。

注意：如果要在 pebblo 的默认 URL（localhost:8000）之外的某个 URL 上配置 pebblo 服务器，请将正确的 URL 放入 `PEBBLO_CLASSIFIER_URL` 环境变量中。也可以使用 `classifier_url` 关键字参数进行配置。参考：[server-configurations](https://daxa-ai.github.io/pebblo/config)

#### 如何启用 Pebblo 文档加载？

假设有一个使用 `CSVLoader` 读取 CSV 文档进行推理的 Langchain RAG 应用程序片段。

以下是使用 `CSVLoader` 进行文档加载的片段。

```python
from langchain_community.document_loaders import CSVLoader
loader = CSVLoader("data/corp_sens_data.csv")
documents = loader.load()
print(documents)
```

只需对上述片段进行少量代码更改，即可启用 Pebblo SafeLoader。

```python
from langchain_community.document_loaders import CSVLoader, PebbloSafeLoader
loader = PebbloSafeLoader(
    CSVLoader("data/corp_sens_data.csv"),
    name="acme-corp-rag-1",  # 应用程序名称（必填）
    owner="Joe Smith",  # 所有者（可选）
    description="支持生产力 RAG 应用程序",  # 描述（可选）
)
documents = loader.load()
print(documents)
```

### 将语义主题和实体发送到 Pebblo 云服务器

要将语义数据发送到 Pebblo 云服务器，请将 api-key 作为参数传递给 PebbloSafeLoader，或者将 api-key 放入 `PEBBLO_API_KEY` 环境变量中。

```python
from langchain_community.document_loaders import CSVLoader, PebbloSafeLoader
loader = PebbloSafeLoader(
    CSVLoader("data/corp_sens_data.csv"),
    name="acme-corp-rag-1",  # 应用程序名称（必填）
    owner="Joe Smith",  # 所有者（可选）
    description="支持生产力 RAG 应用程序",  # 描述（可选）
    api_key="my-api-key",  # API 密钥（可选，也可以在环境变量 PEBBLO_API_KEY 中设置）
)
documents = loader.load()
print(documents)
```

### 将语义主题和实体添加到加载的元数据中

要将语义主题和语义实体添加到加载文档的元数据中，请将 load_semantic 设置为 True 作为参数，或者定义一个新的环境变量 `PEBBLO_LOAD_SEMANTIC`，并将其设置为 True。

```python
from langchain_community.document_loaders import CSVLoader, PebbloSafeLoader
loader = PebbloSafeLoader(
    CSVLoader("data/corp_sens_data.csv"),
    name="acme-corp-rag-1",  # 应用程序名称（必填）
    owner="Joe Smith",  # 所有者（可选）
    description="支持生产力 RAG 应用程序",  # 描述（可选）
    api_key="my-api-key",  # API 密钥（可选，也可以在环境变量 PEBBLO_API_KEY 中设置）
    load_semantic=True,  # 加载语义数据（可选，默认为 False，也可以在环境变量 PEBBLO_LOAD_SEMANTIC 中设置）
)
documents = loader.load()
print(documents[0].metadata)
```