# Bedrock（知识库）

> [亚马逊 Bedrock 的知识库](https://aws.amazon.com/bedrock/knowledge-bases/) 是亚马逊网络服务（AWS）的一项服务，它允许您通过使用私人数据来定制 FM 响应，快速构建 RAG 应用程序。

> 实现 `RAG` 需要组织执行多项繁琐的步骤，将数据转换为嵌入（向量），将嵌入存储在专门的向量数据库中，并构建自定义集成到数据库中以搜索和检索与用户查询相关的文本。这可能是耗时且低效的。

> 使用 `亚马逊 Bedrock 的知识库`，只需指向您在 `亚马逊 S3` 中的数据位置，`亚马逊 Bedrock 的知识库` 就会处理将数据整合到您的向量数据库中的整个摄取工作流程。如果您没有现有的向量数据库，亚马逊 Bedrock 会为您创建一个 Amazon OpenSearch 无服务器向量存储。对于检索，使用 Langchain - 亚马逊 Bedrock 集成通过检索 API 从知识库中检索与用户查询相关的结果。

> 可以通过 [AWS 控制台](https://aws.amazon.com/console/) 或使用 [AWS SDK](https://aws.amazon.com/developer/tools/) 配置知识库。

## 使用知识库检索器

```python
%pip install --upgrade --quiet  boto3
```

```python
from langchain_community.retrievers import AmazonKnowledgeBasesRetriever
retriever = AmazonKnowledgeBasesRetriever(
    knowledge_base_id="PUIJP4EQUA",
    retrieval_config={"vectorSearchConfiguration": {"numberOfResults": 4}},
)
```

```python
query = "总统对 Ketanji Brown 有什么看法？"
retriever.invoke(query)
```

### 在问答链中使用

```python
from botocore.client import Config
from langchain.chains import RetrievalQA
from langchain_community.llms import Bedrock
model_kwargs_claude = {"temperature": 0, "top_k": 10, "max_tokens_to_sample": 3000}
llm = Bedrock(model_id="anthropic.claude-v2", model_kwargs=model_kwargs_claude)
qa = RetrievalQA.from_chain_type(
    llm=llm, retriever=retriever, return_source_documents=True
)
qa(query)
```