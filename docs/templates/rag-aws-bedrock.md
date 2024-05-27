# rag-aws-bedrock

这个模板旨在连接 AWS Bedrock 服务，它是一个提供一组基础模型的托管服务器。

它主要使用 `Anthropic Claude` 进行文本生成，`Amazon Titan` 进行文本嵌入，并利用 FAISS 作为向量存储。

有关 RAG 管道的更多背景信息，请参阅[此笔记本](https://github.com/aws-samples/amazon-bedrock-workshop/blob/main/03_QuestionAnswering/01_qa_w_rag_claude.ipynb)。

## 环境设置

在使用此包之前，请确保已配置 `boto3` 以与您的 AWS 账户配合使用。

有关如何设置和配置 `boto3` 的详细信息，请访问[此页面](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration)。

此外，您需要安装 `faiss-cpu` 包以与 FAISS 向量存储一起使用：

```bash
pip install faiss-cpu
```

您还应设置以下环境变量以反映您的 AWS 配置文件和区域（如果您不使用 `default` AWS 配置文件和 `us-east-1` 区域）：

* `AWS_DEFAULT_REGION`

* `AWS_PROFILE`

## 使用方法

首先，安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一的包：

```shell
langchain app new my-app --package rag-aws-bedrock
```

要将此包添加到现有项目中：

```shell
langchain app add rag-aws-bedrock
```

然后将以下代码添加到您的 `server.py` 文件中：

```python
from rag_aws_bedrock import chain as rag_aws_bedrock_chain
add_routes(app, rag_aws_bedrock_chain, path="/rag-aws-bedrock")
```

（可选）如果您可以访问 LangSmith，则可以配置它以跟踪、监视和调试 LangChain 应用程序。如果您无法访问，请跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将在本地启动一个运行在 [http://localhost:8000](http://localhost:8000) 的 FastAPI 服务器。

您可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 上查看所有模板，并在 [http://127.0.0.1:8000/rag-aws-bedrock/playground](http://127.0.0.1:8000/rag-aws-bedrock/playground) 上访问 playground。

您可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-aws-bedrock")
```