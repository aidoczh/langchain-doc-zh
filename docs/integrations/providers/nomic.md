# Nomic

Nomic目前提供两种产品：

- Atlas：他们的可视化数据引擎

- GPT4All：他们的开源边缘语言模型生态系统

Nomic集成存在于他们自己的[合作伙伴包](https://pypi.org/project/langchain-nomic/)中。您可以使用以下命令安装它：

```python
%pip install -qU langchain-nomic
```

目前，您可以按以下方式导入他们托管的[嵌入模型](/docs/integrations/text_embedding/nomic)：

```python
from langchain_nomic import NomicEmbeddings
```