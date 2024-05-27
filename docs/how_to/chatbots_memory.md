# 如何为聊天机器人添加记忆

聊天机器人的一个关键特性是能够利用先前对话轮的内容作为上下文。这种状态管理可以采用多种形式，包括：

- 简单地将先前的消息填入聊天模型提示中。

- 类似上述方法，但修剪旧消息以减少模型需要处理的分散信息量。

- 更复杂的修改，比如为长时间运行的对话合成摘要。

我们将在下面详细介绍一些技术！

## 设置

您需要安装一些软件包，并将您的 OpenAI API 密钥设置为名为 `OPENAI_API_KEY` 的环境变量：

```python
%pip install --upgrade --quiet langchain langchain-openai
# 设置环境变量 OPENAI_API_KEY 或从 .env 文件加载：
import dotenv
dotenv.load_dotenv()
```

```output
WARNING: You are using pip version 22.0.4; however, version 23.3.2 is available.
You should consider upgrading via the '/Users/jacoblee/.pyenv/versions/3.10.5/bin/python -m pip install --upgrade pip' command.
Note: you may need to restart the kernel to use updated packages.
```

```output
True
```

让我们还设置一个聊天模型，我们将在下面的示例中使用。

```python
from langchain_openai import ChatOpenAI
chat = ChatOpenAI(model="gpt-3.5-turbo-1106")
```

## 消息传递

最简单的记忆形式就是简单地将聊天历史消息传递到链中。以下是一个例子：

```python
from langchain_core.messages import AIMessage, HumanMessage
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. Answer all questions to the best of your ability.",
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)
chain = prompt | chat
chain.invoke(
    {
        "messages": [
            HumanMessage(
                content="Translate this sentence from English to French: I love programming."
            ),
            AIMessage(content="J'adore la programmation."),
            HumanMessage(content="What did you just say?"),
        ],
    }
)
```

```output
AIMessage(content='I said "J\'adore la programmation," which means "I love programming" in French.')
```

我们可以看到，通过将先前的对话传递到链中，它可以将其用作上下文来回答问题。这是支持聊天机器人记忆的基本概念 - 本指南的其余部分将演示传递或重新格式化消息的便利技术。

## 聊天历史

直接将消息存储和传递为数组是完全可以的，但我们也可以使用 LangChain 内置的[消息历史类](https://api.python.langchain.com/en/latest/langchain_api_reference.html#module-langchain.memory)来存储和加载消息。该类的实例负责从持久存储中存储和加载聊天消息。LangChain 与许多提供程序集成 - 您可以在[此处查看集成列表](/docs/integrations/memory) - 但在本演示中，我们将使用一个临时的演示类。

以下是 API 的示例：

```python
from langchain_community.chat_message_histories import ChatMessageHistory
demo_ephemeral_chat_history = ChatMessageHistory()
demo_ephemeral_chat_history.add_user_message(
    "Translate this sentence from English to French: I love programming."
)
demo_ephemeral_chat_history.add_ai_message("J'adore la programmation.")
demo_ephemeral_chat_history.messages
```

```output
[HumanMessage(content='Translate this sentence from English to French: I love programming.'),
 AIMessage(content="J'adore la programmation.")]
```

我们可以直接使用它来为我们的链存储对话轮：

```python
demo_ephemeral_chat_history = ChatMessageHistory()
input1 = "Translate this sentence from English to French: I love programming."
demo_ephemeral_chat_history.add_user_message(input1)
response = chain.invoke(
    {
        "messages": demo_ephemeral_chat_history.messages,
    }
)
demo_ephemeral_chat_history.add_ai_message(response)
input2 = "What did I just ask you?"
demo_ephemeral_chat_history.add_user_message(input2)
chain.invoke(
    {
        "messages": demo_ephemeral_chat_history.messages,
    }
)
```

```output
AIMessage(content='You asked me to translate the sentence "I love programming" from English to French.')
```

## 自动历史管理

前面的示例明确地将消息传递给链。这是一个完全可以接受的方法，但它确实需要对新消息进行外部管理。LangChain 还包括一个名为 `RunnableWithMessageHistory` 的 LCEL 链的包装器，它可以自动处理这个过程。

为了展示它的工作原理，让我们稍微修改上面的提示，以接受一个最终的 `input` 变量，该变量在聊天历史之后填充 `HumanMessage` 模板。这意味着我们将期望一个包含当前消息之前的所有消息的 `chat_history` 参数，而不是所有消息：

```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. Answer all questions to the best of your ability.",
        ),
        MessagesPlaceholder(variable_name="chat_history"),
        ("human", "{input}"),
    ]
)
chain = prompt | chat
```

我们将在这里将最新的输入传递给对话，并让 `RunnableWithMessageHistory` 类包装我们的链条，并完成将该 `input` 变量附加到聊天历史的工作。

接下来，让我们声明我们包装后的链条：

```python
from langchain_core.runnables.history import RunnableWithMessageHistory
demo_ephemeral_chat_history_for_chain = ChatMessageHistory()
chain_with_message_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: demo_ephemeral_chat_history_for_chain,
    input_messages_key="input",
    history_messages_key="chat_history",
)
```

除了要包装的链条之外，这个类还接受一些参数：

- 一个返回给定会话 ID 的消息历史记录的工厂函数。这允许您的链条同时处理多个用户，通过为不同的对话加载不同的消息。

- 一个 `input_messages_key`，指定应该跟踪和存储在聊天历史中的输入的哪个部分。在这个例子中，我们想要跟踪作为 `input` 传递的字符串。

- 一个 `history_messages_key`，指定前面的消息应该注入到提示中。我们的提示有一个名为 `chat_history` 的 `MessagesPlaceholder`，所以我们指定这个属性来匹配。

- （对于具有多个输出的链条）一个 `output_messages_key`，指定要存储为历史记录的输出。这与 `input_messages_key` 相反。

我们可以像正常调用链条一样调用这个新的链条，还可以添加一个 `configurable` 字段，指定要传递给工厂函数的特定 `session_id`。这在演示中未使用，但在实际的链条中，您将希望返回与传递的会话对应的聊天历史记录：

```python
chain_with_message_history.invoke(
    {"input": "Translate this sentence from English to French: I love programming."},
    {"configurable": {"session_id": "unused"}},
)
```

```output
AIMessage(content='The translation of "I love programming" in French is "J\'adore la programmation."')
```

```python
chain_with_message_history.invoke(
    {"input": "What did I just ask you?"},
    {"configurable": {"session_id": "unused"}},
)
```

```output
AIMessage(content='You just asked me to translate the sentence "I love programming" from English to French.')
```

## 修改聊天历史

修改存储的聊天消息可以帮助您的聊天机器人处理各种情况。以下是一些示例：

### 裁剪消息

LLM 和聊天模型有限的上下文窗口，即使您没有直接达到限制，您可能也希望限制模型处理的干扰量。一种解决方案是只加载和存储最近的 `n` 条消息。让我们使用一个带有一些预加载消息的示例历史记录：

```python
demo_ephemeral_chat_history = ChatMessageHistory()
demo_ephemeral_chat_history.add_user_message("Hey there! I'm Nemo.")
demo_ephemeral_chat_history.add_ai_message("Hello!")
demo_ephemeral_chat_history.add_user_message("How are you today?")
demo_ephemeral_chat_history.add_ai_message("Fine thanks!")
demo_ephemeral_chat_history.messages
```

```output
[HumanMessage(content="Hey there! I'm Nemo."),
 AIMessage(content='Hello!'),
 HumanMessage(content='How are you today?'),
 AIMessage(content='Fine thanks!')]
```

让我们将这个消息历史与上面声明的 `RunnableWithMessageHistory` 链条一起使用：

```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. Answer all questions to the best of your ability.",
        ),
        MessagesPlaceholder(variable_name="chat_history"),
        ("human", "{input}"),
    ]
)
chain = prompt | chat
chain_with_message_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: demo_ephemeral_chat_history,
    input_messages_key="input",
    history_messages_key="chat_history",
)
chain_with_message_history.invoke(
    {"input": "What's my name?"},
    {"configurable": {"session_id": "unused"}},
)
```

```output
AIMessage(content='Your name is Nemo.')
```

我们可以看到链条记住了预加载的名字。

但是假设我们有一个非常小的上下文窗口，并且我们想要将传递给链的消息数量减少到最近的2条。我们可以使用 `clear` 方法来删除消息并重新将它们添加到历史记录中。我们不一定要这样做，但让我们将这个方法放在链的最前面，以确保它总是被调用：

```python
from langchain_core.runnables import RunnablePassthrough
def trim_messages(chain_input):
    stored_messages = demo_ephemeral_chat_history.messages
    if len(stored_messages) <= 2:
        return False
    demo_ephemeral_chat_history.clear()
    for message in stored_messages[-2:]:
        demo_ephemeral_chat_history.add_message(message)
    return True
chain_with_trimming = (
    RunnablePassthrough.assign(messages_trimmed=trim_messages)
    | chain_with_message_history
)
```

让我们调用这个新链并检查消息：

```python
chain_with_trimming.invoke(
    {"input": "P. Sherman住在哪里？"},
    {"configurable": {"session_id": "unused"}},
)
```

```output
AIMessage(content="P. Sherman的地址是悉尼42 Wallaby Way。")
```

```python
demo_ephemeral_chat_history.messages
```

```output
[HumanMessage(content="我的名字是什么？"),
 AIMessage(content='你的名字是Nemo。'),
 HumanMessage(content='P. Sherman住在哪里？'),
 AIMessage(content="P. Sherman的地址是悉尼42 Wallaby Way。")]
```

我们可以看到我们的历史记录已经删除了两条最旧的消息，同时在末尾添加了最近的对话。下次调用链时，`trim_messages` 将再次被调用，只有最近的两条消息将被传递给模型。在这种情况下，这意味着下次调用时模型将忘记我们给它的名字：

```python
chain_with_trimming.invoke(
    {"input": "我的名字是什么？"},
    {"configurable": {"session_id": "unused"}},
)
```

```output
AIMessage(content="抱歉，我无法访问您的个人信息。")
```

```python
demo_ephemeral_chat_history.messages
```

```output
[HumanMessage(content='P. Sherman住在哪里？'),
 AIMessage(content="P. Sherman的地址是悉尼42 Wallaby Way。"),
 HumanMessage(content='我的名字是什么？'),
 AIMessage(content="抱歉，我无法访问您的个人信息。")]
```

### 总结记忆

我们也可以以其他方式使用相同的模式。例如，我们可以使用额外的LLM调用来在调用链之前生成对话摘要。让我们重新创建我们的聊天历史和聊天机器人链：

```python
demo_ephemeral_chat_history = ChatMessageHistory()
demo_ephemeral_chat_history.add_user_message("嘿！我是Nemo。")
demo_ephemeral_chat_history.add_ai_message("你好！")
demo_ephemeral_chat_history.add_user_message("你今天好吗？")
demo_ephemeral_chat_history.add_ai_message("谢谢，我很好！")
demo_ephemeral_chat_history.messages
```

```output
[HumanMessage(content="嘿！我是Nemo。"),
 AIMessage(content='你好！'),
 HumanMessage(content='你今天好吗？'),
 AIMessage(content='谢谢，我很好！')]
```

我们将稍微修改提示，让LLM意识到它将收到一个简短摘要而不是聊天历史：

```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "你是一个乐于助人的助手。尽力回答所有问题。提供的聊天历史包括与您交谈的用户的事实。",
        ),
        MessagesPlaceholder(variable_name="chat_history"),
        ("user", "{input}"),
    ]
)
chain = prompt | chat
chain_with_message_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: demo_ephemeral_chat_history,
    input_messages_key="input",
    history_messages_key="chat_history",
)
```

现在，让我们创建一个函数，将之前的交互总结为摘要。我们也可以将这个函数添加到链的最前面：

```python
def summarize_messages(chain_input):
    stored_messages = demo_ephemeral_chat_history.messages
    if len(stored_messages) == 0:
        return False
    summarization_prompt = ChatPromptTemplate.from_messages(
        [
            MessagesPlaceholder(variable_name="chat_history"),
            (
                "user",
                "将上述聊天消息浓缩成一条摘要消息。尽可能包含多个具体细节。",
            ),
        ]
    )
    summarization_chain = summarization_prompt | chat
    summary_message = summarization_chain.invoke({"chat_history": stored_messages})
    demo_ephemeral_chat_history.clear()
    demo_ephemeral_chat_history.add_message(summary_message)
    return True
chain_with_summarization = (
    RunnablePassthrough.assign(messages_summarized=summarize_messages)
    | chain_with_message_history
)
```

让我们看看它是否记得我们给它起的名字：

```python
chain_with_summarization.invoke(
    {"input": "What did I say my name was?"},
    {"configurable": {"session_id": "unused"}},
)
```

输出结果为：

```output
AIMessage(content='你自己介绍过你的名字是 Nemo。我今天能帮你什么吗，Nemo？')
```

查看聊天历史记录：

```python
demo_ephemeral_chat_history.messages
```

输出结果为：

```output
[AIMessage(content='这是 Nemo 和 AI 之间的对话。Nemo 自我介绍后，AI 用问候回应。Nemo 问 AI 近况如何，AI 回答说一切都好。'),
 HumanMessage(content='我自己说过我的名字是什么吗？'),
 AIMessage(content='你自己介绍过你的名字是 Nemo。我今天能帮你什么吗，Nemo？')]
```

请注意，再次调用链式模型会生成一个新的摘要，该摘要包括初始摘要以及新的消息等。您还可以设计一种混合方法，其中一定数量的消息保留在聊天历史记录中，而其他消息则被摘要。