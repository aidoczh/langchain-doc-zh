## Oracle 云基础设施生成式人工智能

Oracle 云基础设施（OCI）生成式人工智能是一项完全托管的服务，提供一组最先进、可定制的大型语言模型（LLMs），涵盖广泛的用例，并可通过单个 API 访问。

使用 OCI 生成式人工智能服务，您可以访问现成的预训练模型，或者基于自己的数据在专用 AI 集群上创建和托管自定义微调模型。该服务和 API 的详细文档可在__[这里](https://docs.oracle.com/en-us/iaas/Content/generative-ai/home.htm)__和__[这里](https://docs.oracle.com/en-us/iaas/api/#/en/generative-ai/20231130/)__找到。

本笔记本将解释如何使用 OCI 的生成式人工智能模型与 LangChain。

### 先决条件

我们需要安装 oci sdk

```python
!pip install -U oci
```

### OCI 生成式人工智能 API 端点

https://inference.generativeai.us-chicago-1.oci.oraclecloud.com

## 认证

此 langchain 集成支持的认证方法有：

1. API 密钥

2. 会话令牌

3. 实例主体

4. 资源主体

这些遵循标准 SDK 认证方法，详细信息请参见__[这里](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdk_authentication_methods.htm)__。

## 用法

```python
from langchain_community.llms import OCIGenAI
# 使用默认的身份验证方法 API 密钥
llm = OCIGenAI(
    model_id="MY_MODEL",
    service_endpoint="https://inference.generativeai.us-chicago-1.oci.oraclecloud.com",
    compartment_id="MY_OCID",
)
response = llm.invoke("Tell me one fact about earth", temperature=0.7)
print(response)
```

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
# 使用会话令牌进行身份验证
llm = OCIGenAI(
    model_id="MY_MODEL",
    service_endpoint="https://inference.generativeai.us-chicago-1.oci.oraclecloud.com",
    compartment_id="MY_OCID",
    auth_type="SECURITY_TOKEN",
    auth_profile="MY_PROFILE",  # 替换为您的配置文件名称
    model_kwargs={"temperature": 0.7, "top_p": 0.75, "max_tokens": 200},
)
prompt = PromptTemplate(input_variables=["query"], template="{query}")
llm_chain = LLMChain(llm=llm, prompt=prompt)
response = llm_chain.invoke("what is the capital of france?")
print(response)
```

```python
from langchain_community.embeddings import OCIGenAIEmbeddings
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
embeddings = OCIGenAIEmbeddings(
    model_id="MY_EMBEDDING_MODEL",
    service_endpoint="https://inference.generativeai.us-chicago-1.oci.oraclecloud.com",
    compartment_id="MY_OCID",
)
vectorstore = FAISS.from_texts(
    [
        "Larry Ellison co-founded Oracle Corporation in 1977 with Bob Miner and Ed Oates.",
        "Oracle Corporation is an American multinational computer technology company headquartered in Austin, Texas, United States.",
    ],
    embedding=embeddings,
)
retriever = vectorstore.as_retriever()
template = """Answer the question based only on the following context:
{context}
Question: {question}
"""
prompt = PromptTemplate.from_template(template)
llm = OCIGenAI(
    model_id="MY_MODEL",
    service_endpoint="https://inference.generativeai.us-chicago-1.oci.oraclecloud.com",
    compartment_id="MY_OCID",
)
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
print(chain.invoke("when was oracle founded?"))
print(chain.invoke("where is oracle headquartered?"))
```