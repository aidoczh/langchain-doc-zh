# SageMakerEndpoint

[Amazon SageMaker](https://aws.amazon.com/sagemaker/) 是一个系统，可以为任何用例构建、训练和部署机器学习（ML）模型，提供完全托管的基础设施、工具和工作流程。

这篇笔记介绍了如何使用托管在 `SageMaker endpoint` 上的 LLM。

```python
!pip3 install langchain boto3
```

## 设置

您需要设置 `SagemakerEndpoint` 调用的以下必需参数：

- `endpoint_name`：部署的 Sagemaker 模型的端点名称。在 AWS 区域内必须是唯一的。

- `credentials_profile_name`：~/.aws/credentials 或 ~/.aws/config 文件中的配置文件名称，其中指定了访问密钥或角色信息。如果未指定，将使用默认凭证配置文件，或者如果在 EC2 实例上，则将使用 IMDS 中的凭证。详情请参阅：https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html

## 示例

```python
from langchain_core.documents import Document
```

```python
example_doc_1 = """
Peter 和 Elizabeth 打车去城里参加夜晚的派对。在派对上，Elizabeth 晕倒了，被紧急送往医院。由于被诊断出脑部受伤，医生告诉 Peter 一定要陪在她身边，直到她康复。因此，Peter 在医院陪伴她3天，没有离开。
"""
docs = [
    Document(
        page_content=example_doc_1,
    )
]
```

## 使用外部 boto3 会话进行初始化的示例

### 用于跨账户场景

```python
import json
from typing import Dict
import boto3
from langchain.chains.question_answering import load_qa_chain
from langchain_community.llms import SagemakerEndpoint
from langchain_community.llms.sagemaker_endpoint import LLMContentHandler
from langchain_core.prompts import PromptTemplate
query = """Elizabeth 住院了多久？
"""
prompt_template = """使用以下上下文片段来回答最后的问题。
{context}
问题：{question}
答案："""
PROMPT = PromptTemplate(
    template=prompt_template, input_variables=["context", "question"]
)
roleARN = "arn:aws:iam::123456789:role/cross-account-role"
sts_client = boto3.client("sts")
response = sts_client.assume_role(
    RoleArn=roleARN, RoleSessionName="CrossAccountSession"
)
client = boto3.client(
    "sagemaker-runtime",
    region_name="us-west-2",
    aws_access_key_id=response["Credentials"]["AccessKeyId"],
    aws_secret_access_key=response["Credentials"]["SecretAccessKey"],
    aws_session_token=response["Credentials"]["SessionToken"],
)
class ContentHandler(LLMContentHandler):
    content_type = "application/json"
    accepts = "application/json"
    def transform_input(self, prompt: str, model_kwargs: Dict) -> bytes:
        input_str = json.dumps({"inputs": prompt, "parameters": model_kwargs})
        return input_str.encode("utf-8")
    def transform_output(self, output: bytes) -> str:
        response_json = json.loads(output.read().decode("utf-8"))
        return response_json[0]["generated_text"]
content_handler = ContentHandler()
chain = load_qa_chain(
    llm=SagemakerEndpoint(
        endpoint_name="endpoint-name",
        client=client,
        model_kwargs={"temperature": 1e-10},
        content_handler=content_handler,
    ),
    prompt=PROMPT,
)
chain({"input_documents": docs, "question": query}, return_only_outputs=True)
```

```python
import json
from typing import Dict
from langchain.chains.question_answering import load_qa_chain
from langchain_community.llms import SagemakerEndpoint
from langchain_community.llms.sagemaker_endpoint import LLMContentHandler
from langchain_core.prompts import PromptTemplate
query = """Elizabeth 住院了多久？
"""
prompt_template = """使用以下上下文片段来回答最后的问题。
{context}
问题：{question}
答案："""
PROMPT = PromptTemplate(
    template=prompt_template, input_variables=["context", "question"]
)
class ContentHandler(LLMContentHandler):
    content_type = "application/json"
    accepts = "application/json"
    def transform_input(self, prompt: str, model_kwargs: Dict) -> bytes:
        input_str = json.dumps({"inputs": prompt, "parameters": model_kwargs})
        return input_str.encode("utf-8")
    def transform_output(self, output: bytes) -> str:
        response_json = json.loads(output.read().decode("utf-8"))
        return response_json[0]["generated_text"]
content_handler = ContentHandler()
chain = load_qa_chain(
    llm=SagemakerEndpoint(
        endpoint_name="endpoint-name",
        credentials_profile_name="credentials-profile-name",
        region_name="us-west-2",
        model_kwargs={"temperature": 1e-10},
        content_handler=content_handler,
    ),
    prompt=PROMPT,
)
chain({"input_documents": docs, "question": query}, return_only_outputs=True)
```