---

sidebar_label: 腾讯混元

---

# 腾讯混元

>[腾讯的混合模型API](https://cloud.tencent.com/document/product/1729)（`混元API`）实现了对话交流、内容生成、分析和理解，可广泛应用于智能客服、智能营销、角色扮演、广告文案撰写、产品描述、剧本创作、简历生成、文章写作、代码生成、数据分析和内容分析等各种场景。

查看[更多信息](https://cloud.tencent.com/document/product/1729)。

```python
from langchain_community.chat_models import ChatHunyuan
from langchain_core.messages import HumanMessage
```

```python
chat = ChatHunyuan(
    hunyuan_app_id=111111111,
    hunyuan_secret_id="YOUR_SECRET_ID",
    hunyuan_secret_key="YOUR_SECRET_KEY",
)
```

```python
chat(
    [
        HumanMessage(
            content="You are a helpful assistant that translates English to French.Translate this sentence from English to French. I love programming."
        )
    ]
)
```

```output
AIMessage(content="J'aime programmer.")
```

## 对于带有流式处理的ChatHunyuan

```python
chat = ChatHunyuan(
    hunyuan_app_id="YOUR_APP_ID",
    hunyuan_secret_id="YOUR_SECRET_ID",
    hunyuan_secret_key="YOUR_SECRET_KEY",
    streaming=True,
)
```

```python
chat(
    [
        HumanMessage(
            content="You are a helpful assistant that translates English to French.Translate this sentence from English to French. I love programming."
        )
    ]
)
```

```output
AIMessageChunk(content="J'aime programmer.")
```