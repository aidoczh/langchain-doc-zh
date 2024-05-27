# Yuan2.0

[Yuan2.0](https://github.com/IEIT-Yuan/Yuan-2.0) 是由 IEIT System 开发的新一代基础大型语言模型。我们已经发布了三个模型，分别是 Yuan 2.0-102B、Yuan 2.0-51B 和 Yuan 2.0-2B。我们为其他开发者提供了相关的预训练、微调和推理服务脚本。Yuan2.0 基于 Yuan1.0，利用更广泛的高质量预训练数据和指导微调数据集，以增强模型对语义、数学、推理、代码、知识等方面的理解。

这个示例演示了如何使用 LangChain 与 `Yuan2.0`（2B/51B/102B）推理进行文本生成交互。

Yuan2.0 设置了一个推理服务，用户只需请求推理 API 即可获得结果，详细介绍请参见 [Yuan2.0 推理服务器](https://github.com/IEIT-Yuan/Yuan-2.0/blob/main/docs/inference_server.md)。

```python
from langchain.chains import LLMChain
from langchain_community.llms.yuan2 import Yuan2
```

```python
# 用于本地部署的 Yuan2.0 推理服务器的默认推理 API
infer_api = "http://127.0.0.1:8000/yuan"
# 在代理环境中直接访问端点
# import os
# os.environ["no_proxy"]="localhost,127.0.0.1,::1"
yuan_llm = Yuan2(
    infer_api=infer_api,
    max_tokens=2048,
    temp=1.0,
    top_p=0.9,
    use_history=False,
)
# 只有在希望 Yuan2.0 跟踪对话历史并将累积的上下文发送到后端模型 API 以使其具有状态时才打开 use_history。默认情况下它是无状态的。
# llm.use_history = True
```

```python
question = "请介绍一下中国。"
```

```python
print(yuan_llm.invoke(question))
```