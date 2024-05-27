# Volc Engine

本文提供了一个关于如何加载 Volcano Embedding 类的指南。

## API 初始化

要使用基于 [VolcEngine](https://www.volcengine.com/docs/82379/1099455) 的 LLM 服务，您需要初始化以下参数：

您可以选择在环境变量中初始化 AK、SK，或者初始化参数：

```base
export VOLC_ACCESSKEY=XXX
export VOLC_SECRETKEY=XXX
```

```python
"""基本初始化和调用"""
import os
from langchain_community.embeddings import VolcanoEmbeddings
os.environ["VOLC_ACCESSKEY"] = ""
os.environ["VOLC_SECRETKEY"] = ""
embed = VolcanoEmbeddings(volcano_ak="", volcano_sk="")
print("embed_documents 结果：")
res1 = embed.embed_documents(["foo", "bar"])
for r in res1:
    print("", r[:8])
```

```output
embed_documents 结果：
 [0.02929673343896866, -0.009310632012784481, -0.060323506593704224, 0.0031018739100545645, -0.002218986628577113, -0.0023125179577618837, -0.04864659160375595, -2.062115163425915e-05]
 [0.01987231895327568, -0.026041055098176003, -0.08395249396562576, 0.020043574273586273, -0.028862033039331436, 0.004629664588719606, -0.023107370361685753, -0.0342753604054451]
```

```python
print("embed_query 结果：")
res2 = embed.embed_query("foo")
print("", r[:8])
```

```output
embed_query 结果：
 [0.01987231895327568, -0.026041055098176003, -0.08395249396562576, 0.020043574273586273, -0.028862033039331436, 0.004629664588719606, -0.023107370361685753, -0.0342753604054451]
```