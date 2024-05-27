# Nuclia

[Nuclia](https://nuclia.com) 可自动索引来自任何内部和外部来源的非结构化数据，提供优化的搜索结果和生成式答案。它可以处理视频和音频转录、图像内容提取和文档解析。

`Nuclia Understanding API` 文档转换器将文本分割成段落和句子，识别实体，提供文本摘要，并为所有句子生成嵌入。

要使用 Nuclia Understanding API，您需要拥有 Nuclia 账户。您可以在 [https://nuclia.cloud](https://nuclia.cloud) 免费创建一个账户，然后 [创建一个 NUA 密钥](https://docs.nuclia.dev/docs/docs/using/understanding/intro)。

```python
%pip install --upgrade --quiet  protobuf
%pip install --upgrade --quiet  nucliadb-protos
```

```python
import os
os.environ["NUCLIA_ZONE"] = "<YOUR_ZONE>"  # 例如 europe-1
os.environ["NUCLIA_NUA_KEY"] = "<YOUR_API_KEY>"
```

要使用 Nuclia 文档转换器，您需要实例化一个 `NucliaUnderstandingAPI` 工具，并将 `enable_ml` 设置为 `True`：

```python
from langchain_community.tools.nuclia import NucliaUnderstandingAPI
nua = NucliaUnderstandingAPI(enable_ml=True)
```

Nuclia 文档转换器必须以异步模式调用，因此您需要使用 `atransform_documents` 方法：

```python
import asyncio
from langchain_community.document_transformers.nuclia_text_transform import (
    NucliaTextTransformer,
)
from langchain_core.documents import Document
async def process():
    documents = [
        Document(page_content="<TEXT 1>", metadata={}),
        Document(page_content="<TEXT 2>", metadata={}),
        Document(page_content="<TEXT 3>", metadata={}),
    ]
    nuclia_transformer = NucliaTextTransformer(nua)
    transformed_documents = await nuclia_transformer.atransform_documents(documents)
    print(transformed_documents)
asyncio.run(process())
```