# SageMaker

让我们加载 `SageMaker Endpoints Embeddings` 类。如果您在 SageMaker 上托管自己的 Hugging Face 模型，可以使用这个类。

有关如何执行此操作的说明，请参见[这里](https://www.philschmid.de/custom-inference-huggingface-sagemaker)。

**注意**：为了处理批量请求，您需要调整自定义 `inference.py` 脚本中 `predict_fn()` 函数中的返回行：

从

`return {"vectors": sentence_embeddings[0].tolist()}`

改为：

`return {"vectors": sentence_embeddings.tolist()}`。

```python
!pip3 install langchain boto3
```

```python
import json
from typing import Dict, List
from langchain_community.embeddings import SagemakerEndpointEmbeddings
from langchain_community.embeddings.sagemaker_endpoint import EmbeddingsContentHandler
class ContentHandler(EmbeddingsContentHandler):
    content_type = "application/json"
    accepts = "application/json"
    def transform_input(self, inputs: list[str], model_kwargs: Dict) -> bytes:
        """
        将输入转换为可以被 SageMaker 端点消费的字节。
        Args:
            inputs: 输入字符串列表。
            model_kwargs: 要传递给端点的额外关键字参数。
        Returns:
            转换后的字节输入。
        """
        # 示例：inference.py 预期一个带有 "inputs" 键的 JSON 字符串：
        input_str = json.dumps({"inputs": inputs, **model_kwargs})
        return input_str.encode("utf-8")
    def transform_output(self, output: bytes) -> List[List[float]]:
        """
        将来自端点的字节输出转换为嵌入列表。
        Args:
            output: 来自 SageMaker 端点的字节输出。
        Returns:
            转换后的输出 - 嵌入列表
        Note:
            外部列表的长度是输入字符串的数量。
            内部列表的长度是嵌入维度。
        """
        # 示例：inference.py 返回一个带有嵌入列表的 "vectors" 键的 JSON 字符串：
        response_json = json.loads(output.read().decode("utf-8"))
        return response_json["vectors"]
content_handler = ContentHandler()
embeddings = SagemakerEndpointEmbeddings(
    # credentials_profile_name="credentials-profile-name",
    endpoint_name="huggingface-pytorch-inference-2023-03-21-16-14-03-834",
    region_name="us-east-1",
    content_handler=content_handler,
)
# client = boto3.client(
#     "sagemaker-runtime",
#     region_name="us-west-2"
# )
# embeddings = SagemakerEndpointEmbeddings(
#     endpoint_name="huggingface-pytorch-inference-2023-03-21-16-14-03-834",
#     client=client
#     content_handler=content_handler,
# )
```

```python
query_result = embeddings.embed_query("foo")
```

```python
doc_results = embeddings.embed_documents(["foo"])
```

```python
doc_results
```