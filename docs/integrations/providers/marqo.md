# Marqo

本页面介绍如何在 LangChain 中使用 Marqo 生态系统。

### **Marqo 是什么？**

Marqo 是一个张量搜索引擎，使用存储在内存中的 HNSW 索引的嵌入来实现尖端的搜索速度。Marqo 可以通过水平索引分片扩展到亿级文档索引，并允许异步和非阻塞的数据上传和搜索。Marqo 使用来自 PyTorch、Huggingface、OpenAI 等的最新机器学习模型。您可以从预配置模型开始，也可以自己带入模型。内置的 ONNX 支持和转换可实现更快的推断速度和更高的 CPU 和 GPU 吞吐量。

由于 Marqo 包含自己的推断，您的文档可以包含文本和图像的混合，您可以将带有来自其他系统数据的 Marqo 索引带入 langchain 生态系统，而无需担心您的嵌入是否兼容。

Marqo 的部署是灵活的，您可以使用我们的 Docker 镜像自行开始，或者[联系我们了解我们的托管云服务！](https://www.marqo.ai/pricing)

要使用我们的 Docker 镜像在本地运行 Marqo，请参阅[入门指南。](https://docs.marqo.ai/latest/)

## 安装和设置

- 使用 `pip install marqo` 安装 Python SDK

## 封装器

### VectorStore

存在一个围绕 Marqo 索引的封装器，允许您在 vectorstore 框架中使用它们。Marqo 允许您从一系列模型中选择用于生成嵌入，并公开一些预处理配置。

Marqo vectorstore 还可以处理现有的多模型索引，其中您的文档包含图像和文本的混合，有关更多信息，请参阅[我们的文档](https://docs.marqo.ai/latest/#multi-modal-and-cross-modal-search)。请注意，使用现有的多模型索引实例化 Marqo vectorstore 将禁用通过 langchain vectorstore `add_texts` 方法向其添加任何新文档的功能。

要导入此 vectorstore：

```python
from langchain_community.vectorstores import Marqo
```

有关 Marqo 封装器及其一些独特功能的更详细演示，请参阅[此笔记本](/docs/integrations/vectorstores/marqo)。