---

sidebar_label: 阿里云PAI EAS

---

# 阿里云PAI EAS

[阿里云PAI（AI平台）](https://www.alibabacloud.com/help/en/pai/?spm=a2c63.p38356.0.0.c26a426ckrxUwZ)是一个轻量级且成本效益高的机器学习平台，采用原生云技术。它为您提供端到端的建模服务，可加速基于数百亿特征和超过100个场景中数百亿样本的模型训练。

[阿里云机器学习平台](https://www.alibabacloud.com/help/en/machine-learning-platform-for-ai/latest/what-is-machine-learning-pai)是面向企业和开发人员的机器学习或深度学习工程平台。它提供易于使用、成本效益高、高性能、易于扩展的插件，可应用于各种行业场景。具有超过140种内置优化算法，`机器学习平台`提供全流程的AI工程能力，包括数据标注（`PAI-iTAG`）、模型构建（`PAI-Designer`和`PAI-DSW`）、模型训练（`PAI-DLC`）、编译优化和推理部署（`PAI-EAS`）。

`PAI-EAS`支持不同类型的硬件资源，包括CPU和GPU，并具有高吞吐量和低延迟。它允许您通过几次点击部署大规模复杂模型，并实时执行弹性的缩放和扩展。还提供全面的运维和监控系统。

## 设置EAS服务

设置环境变量以初始化EAS服务的URL和令牌。

有关更多信息，请使用[此文档](https://www.alibabacloud.com/help/en/pai/user-guide/service-deployment/)。

```bash
export EAS_SERVICE_URL=XXX
export EAS_SERVICE_TOKEN=XXX
```

另一种选项是使用以下代码：

```python
import os
from langchain_community.chat_models import PaiEasChatEndpoint
from langchain_core.language_models.chat_models import HumanMessage
os.environ["EAS_SERVICE_URL"] = "Your_EAS_Service_URL"
os.environ["EAS_SERVICE_TOKEN"] = "Your_EAS_Service_Token"
chat = PaiEasChatEndpoint(
    eas_service_url=os.environ["EAS_SERVICE_URL"],
    eas_service_token=os.environ["EAS_SERVICE_TOKEN"],
)
```

## 运行聊天模型

您可以使用默认设置调用EAS服务，如下所示：

```python
output = chat.invoke([HumanMessage(content="write a funny joke")])
print("output:", output)
```

或者，使用新的推理参数调用EAS服务：

```python
kwargs = {"temperature": 0.8, "top_p": 0.8, "top_k": 5}
output = chat.invoke([HumanMessage(content="write a funny joke")], **kwargs)
print("output:", output)
```

或者，运行流式调用以获取流式响应：

```python
outputs = chat.stream([HumanMessage(content="hi")], streaming=True)
for output in outputs:
    print("stream output:", output)
```