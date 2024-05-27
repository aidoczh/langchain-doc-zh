# Cloudflare Workers AI

[Cloudflare, Inc. (维基百科)](https://en.wikipedia.org/wiki/Cloudflare) 是一家提供内容交付网络服务、云安全服务、DDoS 攻击防护以及 ICANN 认证域名注册服务的美国公司。

[Cloudflare Workers AI](https://developers.cloudflare.com/workers-ai/) 允许您通过 REST API 从您的代码在 `Cloudflare` 网络上运行机器学习模型。

[Cloudflare AI 文档](https://developers.cloudflare.com/workers-ai/models/text-embeddings/) 列出了所有可用的文本嵌入模型。

## 设置

需要 Cloudflare 账户 ID 和 API 令牌。从[此文档](https://developers.cloudflare.com/workers-ai/get-started/rest-api/)中找到如何获取它们。

```python
import getpass
my_account_id = getpass.getpass("输入您的 Cloudflare 账户 ID：\n\n")
my_api_token = getpass.getpass("输入您的 Cloudflare API 令牌：\n\n")
```

## 示例

```python
from langchain_community.embeddings.cloudflare_workersai import (
    CloudflareWorkersAIEmbeddings,
)
```

```python
embeddings = CloudflareWorkersAIEmbeddings(
    account_id=my_account_id,
    api_token=my_api_token,
    model_name="@cf/baai/bge-small-en-v1.5",
)
# 单个字符串嵌入
query_result = embeddings.embed_query("test")
len(query_result), query_result[:3]
```

```output
(384, [-0.033627357333898544, 0.03982774540781975, 0.03559349477291107])
```

```python
# 批量字符串嵌入
batch_query_result = embeddings.embed_documents(["test1", "test2", "test3"])
len(batch_query_result), len(batch_query_result[0])
```

```output
(3, 384)
```