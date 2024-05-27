# Browserbase

[Browserbase](https://browserbase.com) 是一个开发者平台，可可靠地运行、管理和监控无头浏览器。

使用以下功能来提升您的人工智能数据检索能力：

- [无服务器基础设施](https://docs.browserbase.com/under-the-hood)：提供可靠的浏览器，用于从复杂的用户界面中提取数据

- [隐身模式](https://docs.browserbase.com/features/stealth-mode)：包含指纹识别策略和自动验证码解决

- [会话调试器](https://docs.browserbase.com/features/sessions)：可检查您的浏览器会话，包括网络时间轴和日志

- [实时调试](https://docs.browserbase.com/guides/session-debug-connection/browser-remote-control)：快速调试您的自动化流程

## 安装和设置

- 从 [browserbase.com](https://browserbase.com) 获取 API 密钥和项目 ID，并将其设置为环境变量 (`BROWSERBASE_API_KEY`, `BROWSERBASE_PROJECT_ID`)。

- 安装 [Browserbase SDK](http://github.com/browserbase/python-sdk)：

```python
% pip install browserbase
```

## 加载文档

您可以使用 `BrowserbaseLoader` 将网页加载到 LangChain 中。可选择设置 `text_content` 参数，将页面转换为仅文本表示形式。

```python
from langchain_community.document_loaders import BrowserbaseLoader
```

```python
loader = BrowserbaseLoader(
    urls=[
        "https://example.com",
    ],
    # 文本模式
    text_content=False,
)
docs = loader.load()
print(docs[0].page_content[:61])
```

### 加载器选项

- `urls` 必填。要获取的 URL 列表。

- `text_content` 仅检索文本内容。默认为 `False`。

- `api_key` 可选。Browserbase API 密钥。默认为环境变量 `BROWSERBASE_API_KEY`。

- `project_id` 可选。Browserbase 项目 ID。默认为环境变量 `BROWSERBASE_PROJECT_ID`。

- `session_id` 可选。提供现有的会话 ID。

- `proxy` 可选。启用/禁用代理。

## 加载图片

您还可以加载网页的屏幕截图（以字节形式），用于多模态模型。

使用 GPT-4V 的完整示例：

```python
from browserbase import Browserbase
from browserbase.helpers.gpt4 import GPT4VImage, GPT4VImageDetail
from langchain_core.messages import HumanMessage
from langchain_openai import ChatOpenAI
chat = ChatOpenAI(model="gpt-4-vision-preview", max_tokens=256)
browser = Browserbase()
screenshot = browser.screenshot("https://browserbase.com")
result = chat.invoke(
    [
        HumanMessage(
            content=[
                {"type": "text", "text": "What color is the logo?"},
                GPT4VImage(screenshot, GPT4VImageDetail.auto),
            ]
        )
    ]
)
print(result.content)
```