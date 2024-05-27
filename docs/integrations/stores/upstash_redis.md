---

sidebar_label: Upstash Redis

---

# UpstashRedisByteStore

`UpstashRedisStore` 是 `ByteStore` 的一种实现，它将所有数据存储在您的 Upstash 托管的 Redis 实例中。

要使用基本的 `RedisStore`，请参阅[此指南](/docs/integrations/stores/redis/)

要配置 Upstash Redis，请参考我们的[Upstash指南](/docs/integrations/providers/upstash).

```python
%pip install --upgrade --quiet  upstash-redis
```

```python
from langchain_community.storage import UpstashRedisByteStore
from upstash_redis import Redis
URL = "<UPSTASH_REDIS_REST_URL>"
TOKEN = "<UPSTASH_REDIS_REST_TOKEN>"
redis_client = Redis(url=URL, token=TOKEN)
store = UpstashRedisByteStore(client=redis_client, ttl=None, namespace="test-ns")
store.mset([("k1", b"v1"), ("k2", b"v2")])
print(store.mget(["k1", "k2"]))
```

```output
[b'v1', b'v2']
```