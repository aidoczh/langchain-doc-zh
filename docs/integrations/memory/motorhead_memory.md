# Motörhead

[Motörhead](https://github.com/getmetal/motorhead) 是一个用 Rust 实现的内存服务器。它可以自动在后台处理增量摘要，并允许无状态应用程序。

## 设置

请查看[Motörhead](https://github.com/getmetal/motorhead)上的说明，以在本地运行服务器。

```python
from langchain.memory.motorhead_memory import MotorheadMemory
```

## 示例

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
template = """你是一个与人类进行对话的聊天机器人。
{chat_history}
Human: {human_input}
AI:"""
prompt = PromptTemplate(
    input_variables=["chat_history", "human_input"], template=template
)
memory = MotorheadMemory(
    session_id="testing-1", url="http://localhost:8080", memory_key="chat_history"
)
await memory.init()
# 从 Motörhead 加载先前的状态 🤘
llm_chain = LLMChain(
    llm=OpenAI(),
    prompt=prompt,
    verbose=True,
    memory=memory,
)
```

```python
llm_chain.run("hi im bob")
```

```output
> 进入新的 LLMChain 链...
格式化后的提示：
你是一个与人类进行对话的聊天机器人。
Human: hi im bob
AI:
> 链结束。
```

```output
' 嗨，鲍勃，很高兴见到你！你今天过得怎么样？'
```

```python
llm_chain.run("whats my name?")
```

```output
> 进入新的 LLMChain 链...
格式化后的提示：
你是一个与人类进行对话的聊天机器人。
Human: hi im bob
AI:  嗨，鲍勃，很高兴见到你！你今天过得怎么样？
Human: whats my name?
AI:
> 链结束。
```

```output
' 你说你的名字是鲍勃。这正确吗？'
```

```python
llm_chain.run("whats for dinner?")
```

```output
> 进入新的 LLMChain 链...
格式化后的提示：
你是一个与人类进行对话的聊天机器人。
Human: hi im bob
AI:  嗨，鲍勃，很高兴见到你！你今天过得怎么样？
Human: whats my name?
AI:  你说你的名字是鲍勃。这正确吗？
Human: whats for dinner?
AI:
> 链结束。
```

```output
"  对不起，我不确定你在问什么。你能重新表达一下你的问题吗？"
```