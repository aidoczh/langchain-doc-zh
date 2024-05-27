# Nuclia

[Nuclia](https://nuclia.com) 自动索引您来自任何内部和外部来源的非结构化数据，提供优化的搜索结果和生成式答案。它可以处理视频和音频转录，图像内容提取和文档解析。

`Nuclia Understanding API` 支持处理非结构化数据，包括文本、网页、文档和音频/视频内容。它可以提取所有文本内容（在需要时使用语音转文本或 OCR 技术），还可以提取元数据、嵌入式文件（如 PDF 中的图像）和网页链接。如果启用了机器学习，它可以识别实体，提供内容摘要，并为所有句子生成嵌入式向量。

## 设置

要使用 `Nuclia Understanding API`，您需要拥有一个 Nuclia 账户。您可以免费在 [https://nuclia.cloud](https://nuclia.cloud) 创建一个账户，然后 [创建一个 NUA 密钥](https://docs.nuclia.dev/docs/docs/using/understanding/intro)。

```python
%pip install --upgrade --quiet  protobuf
%pip install --upgrade --quiet  nucliadb-protos
```

```python
import os
os.environ["NUCLIA_ZONE"] = "<YOUR_ZONE>"  # 例如：europe-1
os.environ["NUCLIA_NUA_KEY"] = "<YOUR_API_KEY>"
```

## 示例

要使用 Nuclia 文档加载器，您需要实例化一个 `NucliaUnderstandingAPI` 工具：

```python
from langchain_community.tools.nuclia import NucliaUnderstandingAPI
nua = NucliaUnderstandingAPI(enable_ml=False)
```

```python
from langchain_community.document_loaders.nuclia import NucliaLoader
loader = NucliaLoader("./interview.mp4", nua)
```

您现在可以调用 `load` 方法来加载文档，直到获取到文档为止。

```python
import time
pending = True
while pending:
    time.sleep(15)
    docs = loader.load()
    if len(docs) > 0:
        print(docs[0].page_content)
        print(docs[0].metadata)
        pending = False
    else:
        print("waiting...")
```

## 获取的信息

Nuclia 返回以下信息：

- 文件元数据

- 提取的文本

- 嵌套文本（如嵌入图像中的文本）

- 段落和句子拆分（由它们的第一个和最后一个字符的位置定义，以及视频或音频文件的开始时间和结束时间）

- 链接

- 缩略图

- 嵌入文件

注意：

  生成的文件（缩略图、提取的嵌入式文件等）将作为令牌提供。您可以使用 [`/processing/download` 端点](https://docs.nuclia.dev/docs/api#operation/Download_binary_file_processing_download_get) 下载它们。

  此外，在任何级别，如果属性超过一定大小，它将被放入一个可下载的文件中，并将在文档中用文件指针替换。这将包括 `{"file": {"uri": "JWT_TOKEN"}}`。规则是，如果消息的大小超过 1000000 个字符，最大的部分将被移至可下载的文件。首先，压缩过程将针对向量。如果这还不够，它将针对大字段元数据，最后它将针对提取的文本。