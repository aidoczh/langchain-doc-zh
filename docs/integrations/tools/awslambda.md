# AWS Lambda

[`Amazon AWS Lambda`](https://aws.amazon.com/pm/lambda/) 是由 `Amazon Web Services` (`AWS`) 提供的无服务器计算服务。它帮助开发人员构建和运行应用程序和服务，无需预配或管理服务器。这种无服务器架构使您可以专注于编写和部署代码，而 AWS 会自动处理扩展、打补丁和管理运行应用程序所需的基础设施。

本文介绍如何使用 `AWS Lambda` 工具。

通过将 `AWS Lambda` 加入提供给代理的工具列表，您可以授予代理在 AWS 云中调用代码的能力，以满足您的各种需求。

当代理使用 `AWS Lambda` 工具时，它将提供一个字符串类型的参数，该参数将通过事件参数传递给 Lambda 函数。

首先，您需要安装 `boto3` Python 包。

```python
%pip install --upgrade --quiet  boto3 > /dev/null
```

为了使代理能够使用该工具，您必须提供与 Lambda 函数逻辑功能相匹配的名称和描述。

您还必须提供函数的名称。

请注意，由于该工具实际上只是 boto3 库的一个包装器，因此您需要运行 `aws configure` 来使用该工具。有关更多详细信息，请参阅[这里](https://docs.aws.amazon.com/cli/index.html)

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
llm = OpenAI(temperature=0)
tools = load_tools(
    ["awslambda"],
    awslambda_tool_name="email-sender",
    awslambda_tool_description="sends an email with the specified content to test@testing123.com",
    function_name="testFunction1",
)
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
agent.run("Send an email to test@testing123.com saying hello world.")
```