# 泰坦起飞

`TitanML` 帮助企业通过我们的训练、压缩和推理优化平台构建和部署更好、更小、更便宜、更快的自然语言处理模型。

我们的推理服务器，[泰坦起飞](https://docs.titanml.co/docs/intro) 可以通过一条命令在您的硬件上本地部署 LLMs。大多数嵌入模型都可以直接支持，如果您在使用特定模型时遇到问题，请通过 hello@titanml.co 联系我们。

## 示例用法

以下是一些有用的示例，帮助您开始使用泰坦起飞服务器。在运行这些命令之前，请确保 Takeoff 服务器已在后台启动。有关更多信息，请参阅[启动 Takeoff 的文档页面](https://docs.titanml.co/docs/Docs/launching/)。

```python
import time
from langchain_community.embeddings import TitanTakeoffEmbed
```

### 示例 1

基本用法，假设 Takeoff 在您的机器上使用默认端口（即 localhost:3000）运行。

```python
embed = TitanTakeoffEmbed()
output = embed.embed_query(
    "What is the weather in London in August?", consumer_group="embed"
)
print(output)
```

### 示例 2

使用 TitanTakeoffEmbed Python 封装器启动读取器。如果您在首次启动 Takeoff 时没有创建任何读取器，或者想要添加其他读取器，可以在初始化 TitanTakeoffEmbed 对象时执行。只需将要启动的模型列表作为 `models` 参数传递。

您可以使用 `embed.query_documents` 一次嵌入多个文档。预期的输入是一个字符串列表，而不是仅针对 `embed_query` 方法预期的单个字符串。

```python
# 嵌入模型的配置，您可以在此指定以下参数：
#   model_name (str): 要使用的模型名称
#   device: (str): 推理所使用的设备，cuda 或 cpu
#   consumer_group (str): 要将读取器放入的消费者组
embedding_model = {
    "model_name": "BAAI/bge-large-en-v1.5",
    "device": "cpu",
    "consumer_group": "embed",
}
embed = TitanTakeoffEmbed(models=[embedding_model])
# 模型需要时间来启动，所需时间长度取决于模型的大小和您的网络连接速度
time.sleep(60)
prompt = "What is the capital of France?"
# 我们指定了 "embed" 消费者组，因此需要将请求发送到相同的消费者组，以便命中我们的嵌入模型而不是其他模型
output = embed.embed_query(prompt, consumer_group="embed")
print(output)
```