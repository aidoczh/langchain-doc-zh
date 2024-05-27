# 如何配置运行时链内部

:::info 前提条件

本指南假定您熟悉以下概念：

- [LangChain 表达语言 (LCEL)](/docs/concepts/#langchain-expression-language)

- [链接可运行项](/docs/how_to/sequence/)

- [绑定运行时参数](/docs/how_to/binding/)

:::

有时候，您可能希望在链中尝试，甚至向最终用户公开多种不同的操作方式。这可能包括调整参数，比如温度，甚至替换一个模型为另一个。

为了尽可能简化这一体验，我们定义了两种方法。

- `configurable_fields` 方法。这使您能够配置可运行项的特定字段。

  - 这与可运行项的 [`.bind`](/docs/how_to/binding) 方法相关，但允许您在运行时为链中的给定步骤指定参数，而不是事先指定它们。

- `configurable_alternatives` 方法。使用此方法，您可以列出可在运行时设置的任何特定可运行项的替代方案，并将它们替换为指定的替代方案。

## 可配置字段

让我们通过一个示例来演示如何在运行时配置聊天模型字段，比如温度：

```python
%pip install --upgrade --quiet langchain langchain-openai
import os
from getpass import getpass
os.environ["OPENAI_API_KEY"] = getpass()
```

```output
警告：您正在使用 pip 版本 22.0.4；然而，版本 24.0 可用。
您应该考虑通过 '/Users/jacoblee/.pyenv/versions/3.10.5/bin/python -m pip install --upgrade pip' 命令进行升级。
注意：您可能需要重新启动内核才能使用更新后的软件包。
```

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import ConfigurableField
from langchain_openai import ChatOpenAI
model = ChatOpenAI(temperature=0).configurable_fields(
    temperature=ConfigurableField(
        id="llm_temperature",
        name="LLM Temperature",
        description="The temperature of the LLM",
    )
)
model.invoke("pick a random number")
```

```output
AIMessage(content='17', response_metadata={'token_usage': {'completion_tokens': 1, 'prompt_tokens': 11, 'total_tokens': 12}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-ba26a0da-0a69-4533-ab7f-21178a73d303-0')
```

在上面的示例中，我们将 `temperature` 定义为一个 [`ConfigurableField`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.utils.ConfigurableField.html#langchain_core.runnables.utils.ConfigurableField)，我们可以在运行时设置它。为此，我们使用 [`with_config`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.with_config) 方法，如下所示：

```python
model.with_config(configurable={"llm_temperature": 0.9}).invoke("pick a random number")
```

```output
AIMessage(content='12', response_metadata={'token_usage': {'completion_tokens': 1, 'prompt_tokens': 11, 'total_tokens': 12}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-ba8422ad-be77-4cb1-ac45-ad0aae74e3d9-0')
```

请注意，字典中传递的 `llm_temperature` 条目与 `ConfigurableField` 的 `id` 相同。

我们也可以这样做，以影响链中的一个步骤：

```python
prompt = PromptTemplate.from_template("Pick a random number above {x}")
chain = prompt | model
chain.invoke({"x": 0})
```

```output
AIMessage(content='27', response_metadata={'token_usage': {'completion_tokens': 1, 'prompt_tokens': 14, 'total_tokens': 15}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-ecd4cadd-1b72-4f92-b9a0-15e08091f537-0')
```

```python
chain.with_config(configurable={"llm_temperature": 0.9}).invoke({"x": 0})
```

```output
AIMessage(content='35', response_metadata={'token_usage': {'completion_tokens': 1, 'prompt_tokens': 14, 'total_tokens': 15}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-a916602b-3460-46d3-a4a8-7c926ec747c0-0')
```

### 使用 HubRunnables

这对于允许切换提示非常有用：

```python
from langchain.runnables.hub import HubRunnable
prompt = HubRunnable("rlm/rag-prompt").configurable_fields(
    owner_repo_commit=ConfigurableField(
        id="hub_commit",
        name="Hub Commit",
        description="The Hub commit to pull from",
    )
)
prompt.invoke({"question": "foo", "context": "bar"})
```

```output
ChatPromptValue(messages=[HumanMessage(content="You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.\nQuestion: foo \nContext: bar \nAnswer:")])
```

```python
prompt.with_config(configurable={"hub_commit": "rlm/rag-prompt-llama"}).invoke(
    {"question": "foo", "context": "bar"}
)
```

```output
ChatPromptValue(messages=[HumanMessage(content="[INST]<<SYS>> 你是一个用于回答问题的助手。使用以下检索到的上下文片段来回答问题。如果你不知道答案，只需说你不知道。最多使用三句话，保持回答简洁。<</SYS>> \n问题: foo \n上下文: bar \n答案: [/INST]")])
```

## 可配置的替代方案

`configurable_alternatives()` 方法允许我们用替代方案替换链中的步骤。下面，我们将一个聊天模型替换为另一个：

```python
%pip install --upgrade --quiet langchain-anthropic
import os
from getpass import getpass
os.environ["ANTHROPIC_API_KEY"] = getpass()
```

```output
警告: 您正在使用 pip 版本 22.0.4；但本次安装的是 24.0。您可以使用命令 '/Users/jacoblee/.pyenv/versions/3.10.5/bin/python -m pip install --upgrade pip' 来升级 pip。
注意: 您可能需要重新启动内核以使用更新后的软件包。
```

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import ConfigurableField
from langchain_openai import ChatOpenAI
llm = ChatAnthropic(
    model="claude-3-haiku-20240307", temperature=0
).configurable_alternatives(
    # This gives this field an id
    # When configuring the end runnable, we can then use this id to configure this field
    ConfigurableField(id="llm"),
    # This sets a default_key.
    # If we specify this key, the default LLM (ChatAnthropic initialized above) will be used
    default_key="anthropic",
    # This adds a new option, with name `openai` that is equal to `ChatOpenAI()`
    openai=ChatOpenAI(),
    # This adds a new option, with name `gpt4` that is equal to `ChatOpenAI(model="gpt-4")`
    gpt4=ChatOpenAI(model="gpt-4"),
    # You can add more configuration options here
)
prompt = PromptTemplate.from_template("给我讲一个关于{topic}的笑话")
chain = prompt | llm
# 默认情况下会调用 Anthropic
chain.invoke({"topic": "bears"})
```

```output
AIMessage(content="这是一个关于熊的笑话：\n\n为什么熊不穿袜子？\n因为它们有熊脚！\n\n怎么样？我尽量想出一个简单、傻乎乎基于双关语的关于熊的笑话。双关语和言语游戏是创造幽默熊笑话的常用方式。如果你想听另一个，请告诉我！", response_metadata={'id': 'msg_018edUHh5fUbWdiimhrC3dZD', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 13, 'output_tokens': 80}}, id='run-775bc58c-28d7-4e6b-a268-48fa6661f02f-0')
```

```python
# 我们可以使用 `.with_config(configurable={"llm": "openai"})` 来指定要使用的 llm
chain.with_config(configurable={"llm": "openai"}).invoke({"topic": "bears"})
```

```output
AIMessage(content="为什么熊不喜欢快餐？\n\n因为它们抓不到！", response_metadata={'token_usage': {'completion_tokens': 15, 'prompt_tokens': 13, 'total_tokens': 28}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-7bdaa992-19c9-4f0d-9a0c-1f326bc992d4-0')
```

```python
# 如果使用 `default_key`，则使用默认值
chain.with_config(configurable={"llm": "anthropic"}).invoke({"topic": "bears"})
```

```output
AIMessage(content="这是一个关于熊的笑话：\n\n为什么熊不穿袜子？\n因为它们有熊脚！\n\n怎么样？我尽量想出一个简单、傻乎乎基于双关语的关于熊的笑话。双关语和言语游戏是创造幽默熊笑话的常用方式。如果你想听另一个，请告诉我！", response_metadata={'id': 'msg_01BZvbmnEPGBtcxRWETCHkct', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 13, 'output_tokens': 80}}, id='run-59b6ee44-a1cd-41b8-a026-28ee67cdd718-0')
```

### 使用提示

我们可以做类似的事情，但在提示之间交替

```python
llm = ChatAnthropic(model="claude-3-haiku-20240307", temperature=0)
prompt = PromptTemplate.from_template(
    "给我讲一个关于{topic}的笑话"
).configurable_alternatives(
    # This gives this field an id
    # When configuring the end runnable, we can then use this id to configure this field
    ConfigurableField(id="prompt"),
    # This sets a default_key.
    # If we specify this key, the default LLM (ChatAnthropic initialized above) will be used
    default_key="joke",
    # This adds a new option, with name `poem`
    poem=PromptTemplate.from_template("写一首关于{topic}的短诗"),
    # You can add more configuration options here
)
chain = prompt | llm
# 默认情况下会写一个笑话
chain.invoke({"topic": "bears"})
```

```output
AIMessage(content="这里有一个关于熊的笑话：\n\n为什么熊不穿袜子？\n因为它们有熊脚！", response_metadata={'id': 'msg_01DtM1cssjNFZYgeS3gMZ49H', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 13, 'output_tokens': 28}}, id='run-8199af7d-ea31-443d-b064-483693f2e0a1-0')
```

```python
# 我们可以配置它来写一首诗
chain.with_config(configurable={"prompt": "poem"}).invoke({"topic": "熊"})
```

```output
AIMessage(content="这是一首关于熊的短诗：\n\n威严的熊，强大而真实，\n在森林中漫游，野性而自由。\n强大的爪子，柔软而棕色的毛发，\n赢得尊重，大自然的王冠。\n\n觅食浆果，垂钓溪流，\n保护他们的幼崽，凶猛而敏锐。\n强大的熊，令人叹为观止，\n荒野的守护者，无与伦比。\n\n在野外，它们统治着，\n体现了大自然的宏伟主题。\n熊，力量和优雅的象征，\n吸引着所有看到它们的人。", response_metadata={'id': 'msg_01Wck3qPxrjURtutvtodaJFn', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 13, 'output_tokens': 134}}, id='run-69414a1e-51d7-4bec-a307-b34b7d61025e-0')
```

### 使用提示和LLM

我们还可以配置多个可配置项！

以下是使用提示和LLM进行配置的示例。

```python
llm = ChatAnthropic(
    model="claude-3-haiku-20240307", temperature=0
).configurable_alternatives(
    # 这为该字段设置了一个id
    # 当配置最终的可运行项时，我们可以使用该id来配置该字段
    ConfigurableField(id="llm"),
    # 这设置了一个默认键。
    # 如果我们指定了这个键，将使用默认的LLM（上面初始化的ChatAnthropic）
    default_key="anthropic",
    # 这添加了一个新选项，名称为`openai`，等于`ChatOpenAI()`
    openai=ChatOpenAI(),
    # 这添加了一个新选项，名称为`gpt4`，等于`ChatOpenAI(model="gpt-4")`
    gpt4=ChatOpenAI(model="gpt-4"),
    # 您可以在这里添加更多的配置选项
)
prompt = PromptTemplate.from_template(
    "告诉我一个关于{topic}的笑话"
).configurable_alternatives(
    # 这为该字段设置了一个id
    # 当配置最终的可运行项时，我们可以使用该id来配置该字段
    ConfigurableField(id="prompt"),
    # 这设置了一个默认键。
    # 如果我们指定了这个键，将使用默认的LLM（上面初始化的ChatAnthropic）
    default_key="joke",
    # 这添加了一个新选项，名称为`poem`
    poem=PromptTemplate.from_template("写一首关于{topic}的短诗"),
    # 您可以在这里添加更多的配置选项
)
chain = prompt | llm
# 我们可以配置它来写一首关于熊的诗，使用OpenAI
chain.with_config(configurable={"prompt": "poem", "llm": "openai"}).invoke(
    {"topic": "熊"}
)
```

```output
AIMessage(content="在广阔而深邃的森林中，\n熊以优雅和自豪漫游。\n毛发黑如夜，\n以无与伦比的力量统治着土地。\n\n在寒冷的冬天，它们冬眠，\n在春天，它们饥肠辘辘地出现。\n锋利的爪子和敏锐的眼睛，\n它们猎食，凶猛而瘦削。\n\n但在坚韧的外表之下，\n隐藏着一颗温柔的心，温暖而高贵。\n它们全心全意地爱着自己的幼崽，\n日夜保护着它们。\n\n让我们敬佩这些威严的生物，\n对它们的力量和特征感到惊叹。\n因为在野外，它们统治着，\n强大的熊，永恒的梦想。", response_metadata={'token_usage': {'completion_tokens': 133, 'prompt_tokens': 13, 'total_tokens': 146}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-5eec0b96-d580-49fd-ac4e-e32a0803b49b-0')
```

```python
# 如果我们只想配置一个，也可以这样做
chain.with_config(configurable={"llm": "openai"}).invoke({"topic": "熊"})
```

```output
AIMessage(content="为什么熊不穿鞋？\n\n因为它们有熊脚！", response_metadata={'token_usage': {'completion_tokens': 13, 'prompt_tokens': 13, 'total_tokens': 26}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-c1b14c9c-4988-49b8-9363-15bfd479973a-0')
```

### 保存配置

我们还可以轻松地将配置好的链保存为自己的对象

```python
openai_joke = chain.with_config(configurable={"llm": "openai"})
openai_joke.invoke({"topic": "熊"})
```

```output
AIMessage(content="熊为什么和女朋友分手？\n\n因为它再也无法忍受这段关系了！", response_metadata={'token_usage': {'completion_tokens': 20, 'prompt_tokens': 13, 'total_tokens': 33}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-391ebd55-9137-458b-9a11-97acaff6a892-0')
```

## 下一步

您现在已经知道如何在运行时配置链的内部步骤。

要了解更多信息，请参阅本节中关于可运行对象的其他操作指南，包括：

- 使用 [.bind()](/docs/how_to/binding) 作为设置可运行对象的运行时参数的简化方法