# SambaNova

[SambaNova](https://sambanova.ai/)的[SambaStudio](https://sambanova.ai/technology/full-stack-ai-platform)是一个用于运行您自己的开源模型的平台。

以下示例介绍了如何使用LangChain与SambaNova嵌入模型进行交互。

## SambaStudio

**SambaStudio**允许您训练、运行批量推断作业，并部署在线推断端点以运行您自己微调的开源模型。

部署模型需要SambaStudio环境。在[sambanova.ai/products/enterprise-ai-platform-sambanova-suite](https://sambanova.ai/products/enterprise-ai-platform-sambanova-suite)获取更多信息。

注册您的环境变量：

```python
import os
sambastudio_base_url = "<您的SambaStudio环境URL>"
sambastudio_project_id = "<您的SambaStudio项目ID>"
sambastudio_endpoint_id = "<您的SambaStudio端点ID>"
sambastudio_api_key = "<您的SambaStudio端点API密钥>"
# 设置环境变量
os.environ["SAMBASTUDIO_EMBEDDINGS_BASE_URL"] = sambastudio_base_url
os.environ["SAMBASTUDIO_EMBEDDINGS_PROJECT_ID"] = sambastudio_project_id
os.environ["SAMBASTUDIO_EMBEDDINGS_ENDPOINT_ID"] = sambastudio_endpoint_id
os.environ["SAMBASTUDIO_EMBEDDINGS_API_KEY"] = sambastudio_api_key
```

直接从LangChain调用SambaStudio托管的嵌入！

```python
from langchain_community.embeddings.sambanova import SambaStudioEmbeddings
embeddings = SambaStudioEmbeddings()
text = "你好，这是一个测试"
result = embeddings.embed_query(text)
print(result)
texts = ["你好，这是一个测试", "你好，这是另一个测试"]
results = embeddings.embed_documents(texts)
print(results)
```