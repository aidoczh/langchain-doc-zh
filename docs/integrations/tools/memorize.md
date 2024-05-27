# 记忆

使用无监督学习来微调 LLM 本身以记忆信息。

这个工具需要支持微调的 LLM。目前只支持 `langchain.llms import GradientLLM`。

## 导入

```python
import os
from langchain.agents import AgentExecutor, AgentType, initialize_agent, load_tools
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory
from langchain_community.llms import GradientLLM
```

## 设置环境 API 密钥

确保从 Gradient AI 获取您的 API 密钥。您将获得价值 10 美元的免费信用额度，用于测试和微调不同的模型。

```python
from getpass import getpass
if not os.environ.get("GRADIENT_ACCESS_TOKEN", None):
    # 在 https://auth.gradient.ai/select-workspace 下获取访问令牌
    os.environ["GRADIENT_ACCESS_TOKEN"] = getpass("gradient.ai 访问令牌:")
if not os.environ.get("GRADIENT_WORKSPACE_ID", None):
    # `$ gradient workspace list` 中列出的 `ID`
    # 也在登录后在 https://auth.gradient.ai/select-workspace 中显示
    os.environ["GRADIENT_WORKSPACE_ID"] = getpass("gradient.ai 工作空间 ID:")
if not os.environ.get("GRADIENT_MODEL_ADAPTER_ID", None):
    # 在 `$ gradient model list --workspace-id "$GRADIENT_WORKSPACE_ID"` 中列出的 `ID`
    os.environ["GRADIENT_MODEL_ID"] = getpass("gradient.ai 模型 ID:")
```

可选：验证您的环境变量 ```GRADIENT_ACCESS_TOKEN``` 和 ```GRADIENT_WORKSPACE_ID``` 以获取当前部署的模型。

## 创建 `GradientLLM` 实例

您可以指定不同的参数，如模型名称、生成的最大标记、温度等。

```python
llm = GradientLLM(
    model_id=os.environ["GRADIENT_MODEL_ID"],
    # # 可选: 设置新的凭据，它们默认为环境变量
    # gradient_workspace_id=os.environ["GRADIENT_WORKSPACE_ID"],
    # gradient_access_token=os.environ["GRADIENT_ACCESS_TOKEN"],
)
```

## 加载工具

```python
tools = load_tools(["memorize"], llm=llm)
```

## 初始化代理

```python
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    # memory=ConversationBufferMemory(memory_key="chat_history", return_messages=True),
)
```

## 运行代理

要求代理记忆一段文字。

```python
agent.run(
    "请详细记住这个事实：\nZara Tubikova 凭借惊人的技巧，在不到 20 秒的时间内，仅使用双脚解开了一个 4x4 魔方的变种，创造了世界纪录。"
)
```

```output
> 进入新的 AgentExecutor 链...
我应该记住这个事实。
动作: 记忆
动作输入: Zara T
观察: 训练完成。损失: 1.6853971333333335
想法: 我现在知道最终答案了。
最终答案: Zara Tubikova set a world
> 链结束。
```

```output
'Zara Tubikova set a world'
```