# JinaChat

本文介绍了如何开始使用 JinaChat 聊天模型。

```python
from langchain_community.chat_models import JinaChat
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_core.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    SystemMessagePromptTemplate,
)
```

```python
chat = JinaChat(temperature=0)
```

```python
messages = [
    SystemMessage(
        content="You are a helpful assistant that translates English to French."
    ),
    HumanMessage(
        content="Translate this sentence from English to French. I love programming."
    ),
]
chat(messages)
```

```output
AIMessage(content="J'aime programmer.", additional_kwargs={}, example=False)
```

您可以使用 `MessagePromptTemplate` 进行模板化。您可以从一个或多个 `MessagePromptTemplates` 构建 `ChatPromptTemplate`。您可以使用 `ChatPromptTemplate` 的 `format_prompt` 方法，该方法返回一个 `PromptValue`，您可以将其转换为字符串或消息对象，具体取决于您是否希望将格式化值用作 llm 或 chat 模型的输入。

为了方便起见，模板上公开了一个 `from_template` 方法。如果您要使用此模板，它将如下所示：

```python
template = "You are a helpful assistant that translates {input_language} to {output_language}."
system_message_prompt = SystemMessagePromptTemplate.from_template(template)
human_template = "{text}"
human_message_prompt = HumanMessagePromptTemplate.from_template(human_template)
```

```python
chat_prompt = ChatPromptTemplate.from_messages(
    [system_message_prompt, human_message_prompt]
)
# get a chat completion from the formatted messages
chat(
    chat_prompt.format_prompt(
        input_language="English", output_language="French", text="I love programming."
    ).to_messages()
)
```

```output
AIMessage(content="J'aime programmer.", additional_kwargs={}, example=False)
```