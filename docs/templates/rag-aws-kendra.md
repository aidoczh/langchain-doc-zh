# rag-aws-kendra

这个模板是一个应用程序，利用了亚马逊 Kendra，这是一个由机器学习驱动的搜索服务，以及 Anthropic Claude 用于文本生成。该应用程序使用一个检索链来检索文档，以回答来自您的文档的问题。

它使用 `boto3` 库与 Bedrock 服务连接。

要了解如何使用亚马逊 Kendra 构建 RAG 应用程序的更多背景信息，请查看[此页面](https://aws.amazon.com/blogs/machine-learning/quickly-build-high-accuracy-generative-ai-applications-on-enterprise-data-using-amazon-kendra-langchain-and-large-language-models/)。

## 环境设置

请确保设置和配置 `boto3` 以与您的 AWS 帐户配合使用。

您可以按照[此指南](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration)进行操作。

在使用此模板之前，您还应该设置好 Kendra 索引。

您可以使用[此 Cloudformation 模板](https://github.com/aws-samples/amazon-kendra-langchain-extensions/blob/main/kendra_retriever_samples/kendra-docs-index.yaml)创建一个示例索引。

其中包含了包含亚马逊 Kendra、亚马逊 Lex 和亚马逊 SageMaker 的 AWS 在线文档的示例数据。或者，如果您已经对自己的数据集建立了索引，也可以使用您自己的亚马逊 Kendra 索引。

需要设置以下环境变量：

* `AWS_DEFAULT_REGION` - 这应该反映正确的 AWS 区域。默认为 `us-east-1`。

* `AWS_PROFILE` - 这应该反映您的 AWS 配置文件。默认为 `default`。

* `KENDRA_INDEX_ID` - 这应该是 Kendra 索引的索引 ID。请注意，索引 ID 是一个包含 36 个字母数字字符的值，可以在索引详细信息页面找到。

## 使用方法

要使用此软件包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此软件包安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package rag-aws-kendra
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-aws-kendra
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_aws_kendra.chain import chain as rag_aws_kendra_chain
add_routes(app, rag_aws_kendra_chain, path="/rag-aws-kendra")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[此处](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，那么您可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动一个在本地运行的服务器的 FastAPI 应用程序，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/rag-aws-kendra/playground](http://127.0.0.1:8000/rag-aws-kendra/playground)访问 playground。

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-aws-kendra")
```