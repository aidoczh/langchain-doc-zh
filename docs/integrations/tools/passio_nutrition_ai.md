# 快速入门

为了更好地理解 NutritionAI 如何赋予你的代理人超级食品营养能力，让我们构建一个能够通过 Passio NutritionAI 查找信息的代理人。

## 定义工具

我们首先需要创建 [Passio NutritionAI 工具](/docs/integrations/tools/passio_nutrition_ai)。

### [Passio Nutrition AI](/docs/integrations/tools/passio_nutrition_ai)

我们在 LangChain 中内置了一个工具，可以轻松使用 Passio NutritionAI 查找食品营养信息。

请注意，这需要一个 API 密钥 - 他们有免费套餐。

创建 API 密钥后，您需要将其导出为：

```bash
export NUTRITIONAI_SUBSCRIPTION_KEY="..."
```

... 或者通过其他方式（例如 `dotenv` 包）将其提供给您的 Python 环境。您也可以通过构造函数调用来明确控制密钥。

```python
from dotenv import load_dotenv
from langchain_core.utils import get_from_env
load_dotenv()
nutritionai_subscription_key = get_from_env(
    "nutritionai_subscription_key", "NUTRITIONAI_SUBSCRIPTION_KEY"
)
```

```python
from langchain_community.tools.passio_nutrition_ai import NutritionAI
from langchain_community.utilities.passio_nutrition_ai import NutritionAIAPI
```

```python
nutritionai_search = NutritionAI(api_wrapper=NutritionAIAPI())
```

```python
nutritionai_search.invoke("chicken tikka masala")
```

```python
nutritionai_search.invoke("Schnuck Markets sliced pepper jack cheese")
```

### 工具

现在我们有了工具，我们可以创建一个我们将在下游使用的工具列表。

```python
tools = [nutritionai_search]
```

## 创建代理人

现在我们已经定义了工具，我们可以创建代理人。我们将使用 OpenAI Functions 代理人 - 有关此类代理人以及其他选项的更多信息，请参阅[此指南](/docs/concepts#agents)。

首先，我们选择要指导代理人的 LLM。

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
```

接下来，我们选择要用于指导代理人的提示。

```python
from langchain import hub
# 获取要使用的提示 - 您可以修改这个！
prompt = hub.pull("hwchase17/openai-functions-agent")
prompt.messages
```

```output
[SystemMessagePromptTemplate(prompt=PromptTemplate(input_variables=[], template='You are a helpful assistant')),
 MessagesPlaceholder(variable_name='chat_history', optional=True),
 HumanMessagePromptTemplate(prompt=PromptTemplate(input_variables=['input'], template='{input}')),
 MessagesPlaceholder(variable_name='agent_scratchpad')]
```

现在，我们可以使用 LLM、提示和工具初始化代理人。代理人负责接收输入并决定采取什么行动。至关重要的是，代理人不执行这些操作 - 这是由 AgentExecutor（下一步）执行的。有关如何思考这些组件的更多信息，请参阅我们的[概念指南](/docs/concepts#agents)。

```python
from langchain.agents import create_openai_functions_agent
agent = create_openai_functions_agent(llm, tools, prompt)
```

最后，我们将代理人（大脑）与工具结合在 AgentExecutor 中（该执行器将重复调用代理人并执行工具）。有关如何思考这些组件的更多信息，请参阅我们的[概念指南](/docs/concepts#agents)。

```python
from langchain.agents import AgentExecutor
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

## 运行代理人

现在我们可以在几个查询上运行代理人！请注意，目前这些都是**无状态**查询（它不会记住先前的交互）。

```python
agent_executor.invoke({"input": "hi!"})
```

```output
> 进入新的 AgentExecutor 链...
你好！我今天能为您做些什么？
> 链结束。
```

```python
{'input': 'hi!', 'output': 'Hello! How can I assist you today?'}
```

```python
agent_executor.invoke({"input": "how many calories are in a slice pepperoni pizza?"})
```

如果我们想自动跟踪这些消息，我们可以将其包装在 RunnableWithMessageHistory 中。有关如何使用此功能的更多信息，请参阅[此指南](/docs/how_to/message_history)。

```python
agent_executor.invoke(
    {"input": "I had bacon and eggs for breakfast.  How many calories is that?"}
)
```

```python
agent_executor.invoke(
    {
        "input": "I had sliced pepper jack cheese for a snack.  How much protein did I have?"
    }
)
```

```python
agent_executor.invoke(
    {
        "input": "I had sliced colby cheese for a snack. Give me calories for this Schnuck Markets product."
    }
)
```

```python
agent_executor.invoke(
    {
        "input": "I had chicken tikka masala for dinner.  how much calories, protein, and fat did I have with default quantity?"
    }
)
```

## 结论

就这样！在这个快速入门中，我们介绍了如何创建一个简单的代理人，能够将食品营养信息融入到其答案中。代理人是一个复杂的话题，有很多东西需要学习！