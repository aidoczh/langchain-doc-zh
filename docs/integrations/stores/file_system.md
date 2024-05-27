---

sidebar_label: 本地文件系统

sidebar_position: 3

---

# 本地文件存储

`LocalFileStore` 是 `ByteStore` 的一个持久化实现，它将所有数据存储在您选择的文件夹中。

```python
from pathlib import Path
from langchain.storage import LocalFileStore
root_path = Path.cwd() / "data"  # 也可以是一个由字符串设置的路径
store = LocalFileStore(root_path)
store.mset([("k1", b"v1"), ("k2", b"v2")])
print(store.mget(["k1", "k2"]))
```

```output
[b'v1', b'v2']
```

现在让我们看看我们的 `data` 文件夹中存在哪些文件：

```python
!ls {root_path}
```

```output
k1 k2
```