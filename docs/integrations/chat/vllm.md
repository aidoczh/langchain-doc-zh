# vLLM 聊天

vLLM 可以部署为一个模仿 OpenAI API 协议的服务器。这使得 vLLM 可以作为使用 OpenAI API 的应用程序的替代品。可以以与 OpenAI API 相同的格式查询该服务器。

本文介绍如何使用 langchain 的 `ChatOpenAI` 来开始使用 vLLM 聊天模型，**就像它本来就是这样的**。

```python
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_core.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    SystemMessagePromptTemplate,
)
from langchain_openai import ChatOpenAI
```

```python
inference_server_url = "http://localhost:8000/v1"
chat = ChatOpenAI(
    model="mosaicml/mpt-7b",
    openai_api_key="EMPTY",
    openai_api_base=inference_server_url,
    max_tokens=5,
    temperature=0,
)
```

```python
messages = [
    SystemMessage(
        content="You are a helpful assistant that translates English to Italian."
    ),
    HumanMessage(
        content="Translate the following sentence from English to Italian: I love programming."
    ),
]
chat(messages)
```

```output
AIMessage(content=' Io amo programmare', additional_kwargs={}, example=False)
```

您可以使用 `MessagePromptTemplate` 来使用模板。您可以从一个或多个 `MessagePromptTemplate` 构建一个 `ChatPromptTemplate`。您可以使用 ChatPromptTemplate 的 `format_prompt` 方法，该方法返回一个 `PromptValue`，您可以将其转换为字符串或 `Message` 对象，具体取决于您是否希望将格式化的值作为输入传递给 llm 或 chat 模型。

为了方便起见，模板上暴露了一个 `from_template` 方法。如果您要使用此模板，它将如下所示：

```python
template = (
    "You are a helpful assistant that translates {input_language} to {output_language}."
)
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
        input_language="English", output_language="Italian", text="I love programming."
    ).to_messages()
)
```

```output
AIMessage(content=' I love programming too.', additional_kwargs={}, example=False)
```