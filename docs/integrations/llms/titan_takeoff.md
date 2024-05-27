# 泰坦起飞

`TitanML` 帮助企业通过我们的训练、压缩和推理优化平台构建和部署更好、更小、更便宜、更快的自然语言处理模型。

我们的推理服务器，[泰坦起飞](https://docs.titanml.co/docs/intro) 可以通过一个命令在您的硬件上本地部署 LLMs。支持大多数生成模型架构，如 Falcon、Llama 2、GPT2、T5 等等。如果您在使用特定模型时遇到问题，请通过 hello@titanml.co 与我们联系。

## 示例用法

以下是一些有用的示例，帮助您开始使用泰坦起飞服务器。在运行这些命令之前，请确保起飞服务器已在后台启动。欲了解更多信息，请参阅[启动起飞的文档页面](https://docs.titanml.co/docs/Docs/launching/)。

```python
import time
# 请注意，导入 TitanTakeoffPro 而不是 TitanTakeoff 也可以正常工作，因为两者在底层使用相同的对象
from langchain_community.llms import TitanTakeoff
from langchain_core.callbacks import CallbackManager, StreamingStdOutCallbackHandler
from langchain_core.prompts import PromptTemplate
```

### 示例 1

基本用法，假设 Takeoff 在您的机器上运行，并使用默认端口（即 localhost:3000）。

```python
llm = TitanTakeoff()
output = llm.invoke("伦敦八月份的天气如何？")
print(output)
```

### 示例 2

指定端口和其他生成参数。

```python
llm = TitanTakeoff(port=3000)
# 可以在 https://docs.titanml.co/docs/next/apis/Takeoff%20inference_REST_API/generate#request 找到参数的详细列表
output = llm.invoke(
    "世界上最大的雨林是哪个？",
    consumer_group="primary",
    min_new_tokens=128,
    max_new_tokens=512,
    no_repeat_ngram_size=2,
    sampling_topk=1,
    sampling_topp=1.0,
    sampling_temperature=1.0,
    repetition_penalty=1.0,
    regex_string="",
    json_schema=None,
)
print(output)
```

### 示例 3

使用 generate 处理多个输入。

```python
llm = TitanTakeoff()
rich_output = llm.generate(["什么是深度学习？", "什么是机器学习？"])
print(rich_output.generations)
```

### 示例 4

流式输出。

```python
llm = TitanTakeoff(
    streaming=True, callback_manager=CallbackManager([StreamingStdOutCallbackHandler()])
)
prompt = "法国的首都是哪里？"
output = llm.invoke(prompt)
print(output)
```

### 示例 5

使用 LCEL。

```python
llm = TitanTakeoff()
prompt = PromptTemplate.from_template("告诉我关于{topic}的信息")
chain = prompt | llm
output = chain.invoke({"topic": "宇宙"})
print(output)
```

### 示例 6

使用泰坦起飞 Python 封装器启动读取器。如果您在首次启动 Takeoff 时尚未创建任何读取器，或者希望在初始化 TitanTakeoff 对象时添加另一个读取器，可以通过将要启动的模型配置列表作为 `models` 参数传递。

```python
# Llama 模型的配置，您可以指定以下参数：
#   model_name (str): 要使用的模型名称
#   device: (str): 推理时要使用的设备，cuda 或 cpu
#   consumer_group (str): 要将读取器放入的消费者组
#   tensor_parallel (Optional[int]): 您希望模型跨越的 GPU 数量
#   max_seq_length (int): 用于推理的最大序列长度，默认为 512
#   max_batch_size (int_: 连续批处理请求的最大批处理大小
llama_model = {
    "model_name": "TheBloke/Llama-2-7b-Chat-AWQ",
    "device": "cuda",
    "consumer_group": "llama",
}
llm = TitanTakeoff(models=[llama_model])
# 模型需要时间来启动，所需时间的长短取决于模型的大小和您的网络连接速度
time.sleep(60)
prompt = "法国的首都是哪里？"
output = llm.invoke(prompt, consumer_group="llama")
print(output)
```