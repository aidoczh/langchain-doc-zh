# 记忆球

本页面介绍如何在 LangChain 中使用 [记忆球](https://remembrall.dev) 生态系统。

## 什么是记忆球？

记忆球可以让你的语言模型拥有长期记忆、检索增强生成，以及通过几行代码实现完全可观察性。

![记忆球仪表盘的屏幕截图，显示请求统计和模型交互。](/img/RemembrallDashboard.png "记忆球仪表盘界面")

它作为一个轻量级代理，简单地在你的 OpenAI 调用之上增加聊天调用的上下文，其中包含已收集的相关事实。

## 设置

要开始使用，[在记忆球平台上使用 Github 登录](https://remembrall.dev/login)，并从[设置页面复制你的 API 密钥](https://remembrall.dev/dashboard/settings)。

通过修改 `openai_api_base`（见下文）和记忆球 API 密钥发送的任何请求将自动在记忆球仪表盘中进行跟踪。你**永远**不需要与我们的平台分享你的 OpenAI 密钥，这些信息也**永远**不会被记忆球系统存储。

为此，我们需要安装以下依赖项：

```bash
pip install -U langchain-openai
```

### 启用长期记忆

除了通过 `x-gp-api-key` 设置 `openai_api_base` 和记忆球 API 密钥外，你还应该指定一个 UID 来维护记忆。这通常会是一个唯一的用户标识符（比如电子邮件）。

```python
from langchain_openai import ChatOpenAI
chat_model = ChatOpenAI(openai_api_base="https://remembrall.dev/api/openai/v1",
                        model_kwargs={
                            "headers":{
                                "x-gp-api-key": "记忆球API密钥在此",
                                "x-gp-remember": "user@email.com",
                            }
                        })
chat_model.predict("我最喜欢的颜色是蓝色。")
import time; time.sleep(5)  # 等待系统通过自动保存保存事实
print(chat_model.predict("我的最喜欢的颜色是什么？"))
```

### 启用检索增强生成

首先，在[记忆球仪表盘](https://remembrall.dev/dashboard/spells)中创建一个文档上下文。粘贴文档文本或上传要处理的 PDF 文档。保存文档上下文 ID，并按照下面的示例插入。

```python
from langchain_openai import ChatOpenAI
chat_model = ChatOpenAI(openai_api_base="https://remembrall.dev/api/openai/v1",
                        model_kwargs={
                            "headers":{
                                "x-gp-api-key": "记忆球API密钥在此",
                                "x-gp-context": "文档上下文ID在此",
                            }
                        })
print(chat_model.predict("这是一个可以用我的文档回答的问题。"))
```