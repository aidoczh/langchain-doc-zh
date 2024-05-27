# AwaDB

[AwaDB](https://github.com/awa-ai/awadb) 是一种用于搜索和存储由LLM应用程序使用的嵌入向量的AI原生数据库。

## 安装和设置

```bash
pip install awadb
```

## 向量存储

```python
from langchain_community.vectorstores import AwaDB
```

查看[使用示例](/docs/integrations/vectorstores/awadb)。

## 嵌入模型

```python
from langchain_community.embeddings import AwaEmbeddings
```

查看[使用示例](/docs/integrations/text_embedding/awadb)。