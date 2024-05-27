## 甲骨文云基础设施生成式人工智能

甲骨文云基础设施（OCI）生成式人工智能是一项完全托管的服务，提供一组最先进的、可定制的大型语言模型（LLMs），涵盖广泛的用例，并可通过单个 API 访问。

使用 OCI 生成式人工智能服务，您可以访问现成的预训练模型，或者基于自己的数据在专用 AI 集群上创建和托管自定义微调模型。服务和 API 的详细文档可在__[这里](https://docs.oracle.com/en-us/iaas/Content/generative-ai/home.htm)__和__[这里](https://docs.oracle.com/en-us/iaas/api/#/en/generative-ai/20231130/)__找到。

本笔记本将解释如何使用 OCI 的生成式 AI 模型与 LangChain。

### 先决条件

我们需要安装 oci sdk

```python
!pip install -U oci
```

### OCI 生成式人工智能 API 端点

https://inference.generativeai.us-chicago-1.oci.oraclecloud.com

## 认证

此 langchain 集成支持的认证方法包括：

1. API 密钥

2. 会话令牌

3. 实例主体

4. 资源主体

这些遵循标准的 SDK 认证方法，详细信息可在__[这里](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdk_authentication_methods.htm)__找到。

## 用法

```python
from langchain_community.embeddings import OCIGenAIEmbeddings
# 使用默认的 API 密钥进行身份验证
embeddings = OCIGenAIEmbeddings(
    model_id="MY_EMBEDDING_MODEL",
    service_endpoint="https://inference.generativeai.us-chicago-1.oci.oraclecloud.com",
    compartment_id="MY_OCID",
)
query = "这是一个英文查询。"
response = embeddings.embed_query(query)
print(response)
documents = ["这是一个示例文档", "这里还有一个"]
response = embeddings.embed_documents(documents)
print(response)
```

```python
# 使用会话令牌进行身份验证
embeddings = OCIGenAIEmbeddings(
    model_id="MY_EMBEDDING_MODEL",
    service_endpoint="https://inference.generativeai.us-chicago-1.oci.oraclecloud.com",
    compartment_id="MY_OCID",
    auth_type="SECURITY_TOKEN",
    auth_profile="MY_PROFILE",  # 用您的配置文件名称替换
)
query = "这是一个示例查询"
response = embeddings.embed_query(query)
print(response)
documents = ["这是一个示例文档", "这里还有一个"]
response = embeddings.embed_documents(documents)
print(response)
```