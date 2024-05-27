# SparkLLM 聊天

SparkLLM 聊天模型 API 由讯飞科技提供。更多信息请参见 [讯飞开放平台](https://www.xfyun.cn/)。

## 基本用法

```python
"""用于基本初始化和调用"""
from langchain_community.chat_models import ChatSparkLLM
from langchain_core.messages import HumanMessage
chat = ChatSparkLLM(
    spark_app_id="<app_id>", spark_api_key="<api_key>", spark_api_secret="<api_secret>"
)
message = HumanMessage(content="你好")
chat([message])
```

```output
AIMessage(content='你好！我能为您做些什么呢？')
```

- 从 [讯飞 SparkLLM API 控制台](https://console.xfyun.cn/services/bm3) 获取 SparkLLM 的 app_id、api_key 和 api_secret（更多信息请参见 [讯飞 SparkLLM 简介](https://xinghuo.xfyun.cn/sparkapi)），然后设置环境变量 `IFLYTEK_SPARK_APP_ID`、`IFLYTEK_SPARK_API_KEY` 和 `IFLYTEK_SPARK_API_SECRET`，或者像上面的示例一样在创建 `ChatSparkLLM` 时传递参数。

## 用于带流式处理的 ChatSparkLLM

```python
chat = ChatSparkLLM(
    spark_app_id="<app_id>",
    spark_api_key="<api_key>",
    spark_api_secret="<api_secret>",
    streaming=True,
)
for chunk in chat.stream("你好！"):
    print(chunk.content, end="")
```

```output
你好！我能为您做些什么呢？
```

## 对于 v2

```python
"""用于基本初始化和调用"""
from langchain_community.chat_models import ChatSparkLLM
from langchain_core.messages import HumanMessage
chat = ChatSparkLLM(
    spark_app_id="<app_id>",
    spark_api_key="<api_key>",
    spark_api_secret="<api_secret>",
    spark_api_url="wss://spark-api.xf-yun.com/v2.1/chat",
    spark_llm_domain="generalv2",
)
message = HumanMessage(content="你好")
chat([message])
```