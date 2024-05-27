---

sidebar_label: Redis

---

# RedisStore

`RedisStore` 是 `ByteStore` 的一种实现，它将所有数据存储在您的 Redis 实例中。

要配置 Redis，请参考我们的[Redis指南](/docs/integrations/providers/redis)。

```python
%pip install --upgrade --quiet  redis
```

```python
from langchain_community.storage import RedisStore
store = RedisStore(redis_url="redis://localhost:6379")
store.mset([("k1", b"v1"), ("k2", b"v2")])
print(store.mget(["k1", "k2"]))
```

```output
[b'v1', b'v2']
```