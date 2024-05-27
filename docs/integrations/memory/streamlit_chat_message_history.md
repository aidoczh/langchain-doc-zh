# Streamlit

>[Streamlit](https://docs.streamlit.io/) 是一个开源的 Python 库，可以轻松创建和共享漂亮的机器学习和数据科学定制 Web 应用程序。

本文介绍了如何在 `Streamlit` 应用程序中存储和使用聊天消息历史记录。`StreamlitChatMessageHistory` 将消息存储在 [Streamlit 会话状态](https://docs.streamlit.io/library/api-reference/session-state) 中的指定 `key=` 中。默认的 key 是 `"langchain_messages"`。

- 注意，`StreamlitChatMessageHistory` 仅在 Streamlit 应用程序中运行时才有效。

- 您可能还对 LangChain 的 [StreamlitCallbackHandler](/docs/integrations/callbacks/streamlit) 感兴趣。

- 欲了解更多关于 Streamlit 的信息，请查看他们的 [入门文档](https://docs.streamlit.io/library/get-started)。

集成位于 `langchain-community` 包中，因此我们需要安装该包。我们还需要安装 `streamlit`。

```
pip install -U langchain-community streamlit
```

您可以在[此处查看完整的应用程序示例](https://langchain-st-memory.streamlit.app/)，以及在 [github.com/langchain-ai/streamlit-agent](https://github.com/langchain-ai/streamlit-agent) 中的更多示例。

```python
from langchain_community.chat_message_histories import (
    StreamlitChatMessageHistory,
)
history = StreamlitChatMessageHistory(key="chat_messages")
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

```python
history.messages
```

我们可以很容易地将此消息历史记录类与 [LCEL Runnables](/docs/how_to/message_history) 结合使用。

在给定用户会话中，该历史记录将在重新运行 Streamlit 应用程序时保持不变。给定的 `StreamlitChatMessageHistory` 不会在用户会话之间持久化或共享。

```python
# 可选地，为存储消息指定自己的 session_state key
msgs = StreamlitChatMessageHistory(key="special_app_key")
if len(msgs.messages) == 0:
    msgs.add_ai_message("How can I help you?")
```

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are an AI chatbot having a conversation with a human."),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)
chain = prompt | ChatOpenAI()
```

```python
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: msgs,  # 始终返回之前创建的实例
    input_messages_key="question",
    history_messages_key="history",
)
```

在每次重新运行时，对话式 Streamlit 应用程序通常会重新绘制每条先前的聊天消息。通过迭代 `StreamlitChatMessageHistory.messages`，这很容易实现：

```python
import streamlit as st
for msg in msgs.messages:
    st.chat_message(msg.type).write(msg.content)
if prompt := st.chat_input():
    st.chat_message("human").write(prompt)
    # 与往常一样，当调用 Chain 时，新消息将添加到 StreamlitChatMessageHistory 中。
    config = {"configurable": {"session_id": "any"}}
    response = chain_with_history.invoke({"question": prompt}, config)
    st.chat_message("ai").write(response.content)
```

**[查看最终应用程序](https://langchain-st-memory.streamlit.app/)**。