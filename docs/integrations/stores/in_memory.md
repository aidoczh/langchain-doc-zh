---

侧边栏标签: 内存

侧边栏位置: 2

关键词: [内存存储]

---

# 内存字节存储

`InMemoryByteStore` 是 `ByteStore` 的非持久化实现，它将所有数据存储在 Python 字典中。

```python
from langchain.storage import InMemoryByteStore
store = InMemoryByteStore()
store.mset([("k1", b"v1"), ("k2", b"v2")])
print(store.mget(["k1", "k2"]))
```

```output
[b'v1', b'v2']
```