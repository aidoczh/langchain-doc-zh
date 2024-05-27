# 如何添加聊天记录

在许多问答应用程序中，我们希望允许用户进行来回对话，这意味着应用程序需要一些过去问题和答案的“记忆”，以及一些逻辑来将它们融入当前的思考中。

在本指南中，我们专注于**添加逻辑以融入历史消息**。

这在很大程度上是[对话式 RAG 教程](/docs/tutorials/qa_chat_history)的简化版本。

我们将涵盖两种方法：

1. [Chains](/docs/how_to/qa_chat_history_how_to#chains)，其中我们总是执行检索步骤；

2. [Agents](/docs/how_to/qa_chat_history_how_to#agents)，其中我们让一个 LLM 自行决定是否以及如何执行检索步骤（或多个步骤）。

对于外部知识来源，我们将使用相同的[Lilian Weng 的 LLM 动力自主代理](https://lilianweng.github.io/posts/2023-06-23-agent/)博文，来自[RAG 教程](/docs/tutorials/rag)。

## 设置

### 依赖项

在本教程中，我们将使用 OpenAI 嵌入和 Chroma 向量存储，但这里展示的所有内容都适用于任何[嵌入模型](/docs/concepts#embedding-models)和[向量存储](/docs/concepts#vectorstores)或[检索器](/docs/concepts#retrievers)。

我们将使用以下软件包：

```python
%pip install --upgrade --quiet langchain langchain-community langchainhub langchain-chroma bs4
```

我们需要设置环境变量 `OPENAI_API_KEY`，可以直接设置，也可以从 `.env` 文件中加载，如下所示：

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
# import dotenv
# dotenv.load_dotenv()
```

### LangSmith

使用 LangChain 构建的许多应用程序都包含多个步骤，需要多次调用 LLM。随着这些应用程序变得越来越复杂，能够检查链或代理内部发生的情况变得至关重要。最好的方法是使用[LangSmith](https://smith.langchain.com)。

请注意，LangSmith 不是必需的，但很有帮助。如果您想使用 LangSmith，在上面的链接注册后，请确保设置您的环境变量以开始记录跟踪：

```python
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## Chains {#chains}

在对话式 RAG 应用程序中，发给检索器的查询应该受到对话上下文的影响。LangChain 提供了一个[create_history_aware_retriever](https://api.python.langchain.com/en/latest/chains/langchain.chains.history_aware_retriever.create_history_aware_retriever.html)构造函数来简化这一过程。它构建了一个链，接受 `input` 和 `chat_history` 作为输入键，并具有与检索器相同的输出模式。`create_history_aware_retriever`需要以下输入：

1. LLM；

2. 检索器；

3. 提示。

首先，我们获取这些对象：

### LLM

我们可以使用任何支持的聊天模型：

```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs customVarName="llm" />
```

### 检索器

对于检索器，我们将使用[WebBaseLoader](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.web_base.WebBaseLoader.html)来加载网页内容。在这里，我们实例化一个 `Chroma` 向量存储，然后使用其[.as_retriever](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html#langchain_core.vectorstores.VectorStore.as_retriever)方法构建一个可以纳入[LCEL](/docs/concepts/#langchain-expression-language)链的检索器。

```python
import bs4
from langchain import hub
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs=dict(
        parse_only=bs4.SoupStrainer(
            class_=("post-content", "post-title", "post-header")
        )
    ),
)
docs = loader.load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = text_splitter.split_documents(docs)
vectorstore = Chroma.from_documents(documents=splits, embedding=OpenAIEmbeddings())
retriever = vectorstore.as_retriever()
```

### 提示

我们将使用一个包含名为“chat_history”的`MessagesPlaceholder`变量的提示。这样，我们可以使用“chat_history”输入键将一系列消息传递给提示，并且这些消息将被插入到系统消息之后和包含最新问题的人类消息之前。

```python
from langchain.chains import create_history_aware_retriever
from langchain_core.prompts import MessagesPlaceholder
contextualize_q_system_prompt = (
    "给定一个聊天历史记录和最新的用户问题，"
    "该问题可能引用了聊天历史记录中的上下文，"
    "请构造一个可以在没有聊天历史记录的情况下理解的独立问题。"
    "不要回答这个问题，"
    "只需重新表述它（如果需要）并原样返回。"
)
contextualize_q_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", contextualize_q_system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
```

### 组装链条

然后，我们可以实例化具有历史感知的检索器：

```python
history_aware_retriever = create_history_aware_retriever(
    llm, retriever, contextualize_q_prompt
)
```

这个链条在我们的检索器之前添加了输入查询的重新表述，以便检索器能够融入对话的上下文。

现在我们可以构建完整的问答链条。

与[RAG教程](/docs/tutorials/rag)中一样，我们将使用[create_stuff_documents_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.combine_documents.stuff.create_stuff_documents_chain.html)来生成一个`question_answer_chain`，它接受检索到的上下文以及对话历史记录和查询来生成答案。

我们使用[create_retrieval_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.retrieval.create_retrieval_chain.html)构建我们的最终`rag_chain`。这个链条按顺序应用`history_aware_retriever`和`question_answer_chain`，保留中间输出（如检索到的上下文）以方便使用。它的输入键为`input`和`chat_history`，并在输出中包含`input`、`chat_history`、`context`和`answer`。

```python
system_prompt = (
    "你是一个用于问答任务的助手。"
    "使用以下检索到的上下文来回答问题。如果你不知道答案，说你不知道。"
    "最多使用三个句子，回答要简洁。"
    "\n\n"
    "{context}"
)
qa_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
question_answer_chain = create_stuff_documents_chain(llm, qa_prompt)
rag_chain = create_retrieval_chain(history_aware_retriever, question_answer_chain)
```

### 添加聊天历史记录

为了管理聊天历史记录，我们需要：

1. 一个用于存储聊天历史记录的对象；

2. 一个包装我们的链条并管理聊天历史记录更新的对象。

我们将使用[BaseChatMessageHistory](https://api.python.langchain.com/en/latest/chat_history/langchain_core.chat_history.BaseChatMessageHistory.html)和[RunnableWithMessageHistory](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.history.RunnableWithMessageHistory.html)来实现这些功能。后者是一个包装器，用于LCEL链条和`BaseChatMessageHistory`，它处理将聊天历史记录注入到输入中并在每次调用后更新它。

有关如何使用这些类一起创建有状态的对话链条的详细步骤，请参阅[如何添加消息历史记录（内存）](/docs/how_to/message_history/) LCEL指南。

下面，我们实现了第二个选项的一个简单示例，其中聊天历史记录存储在一个简单的字典中。LangChain通过与[Redis](/docs/integrations/memory/redis_chat_message_history/)和其他技术的内存集成来提供更可靠的持久性。

`RunnableWithMessageHistory`的实例会为您管理聊天历史记录。它们接受一个配置，其中包含一个键（默认为“session_id”），指定要获取和添加到输入中的对话历史记录，并将输出附加到相同的对话历史记录中。以下是一个示例：

```python
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
store = {}
def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]
conversational_rag_chain = RunnableWithMessageHistory(
    rag_chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history",
    output_messages_key="answer",
)
```
```python
conversational_rag_chain.invoke(
    {"input": "什么是任务分解?"},
    config={
        "configurable": {"session_id": "abc123"}
    },  # 在`store`中构建一个名为"abc123"的键。
)["answer"]
```
```output
'任务分解涉及将复杂任务分解为更小更简单的步骤，使其更易管理和更容易完成。这个过程可以使用“思维链”（CoT）或“思维树”等技术来引导模型逐步思考或在每一步探索多个推理可能性。任务分解可以通过向语言模型提供简单提示、特定任务说明或人类输入来实现。'
```
```python
conversational_rag_chain.invoke(
    {"input": "常见的做法有哪些?"},
    config={"configurable": {"session_id": "abc123"}},
)["answer"]
```
```output
'任务分解可以通过各种方法实现，包括使用“思维链”（CoT）或“思维树”等技术来引导模型将复杂任务分解为更小的步骤。常见的任务分解方法包括向语言模型提供简单提示、针对特定任务量身定制的任务说明，或者融入人类输入以有效引导分解过程。'
```

对话历史可以在`store`字典中检查：

```python
from langchain_core.messages import AIMessage
for message in store["abc123"].messages:
    if isinstance(message, AIMessage):
        prefix = "AI"
    else:
        prefix = "User"
    print(f"{prefix}: {message.content}\n")
```
```output
User: 什么是任务分解?
AI: 任务分解涉及将复杂任务分解为更小更简单的步骤，使其更易管理和更容易完成。这个过程可以使用“思维链”（CoT）或“思维树”等技术来引导模型逐步思考或在每一步探索多个推理可能性。任务分解可以通过向语言模型提供简单提示、特定任务说明或人类输入来实现。
User: 常见的做法有哪些?
AI: 任务分解可以通过各种方法实现，包括使用“思维链”（CoT）或“思维树”等技术来引导模型将复杂任务分解为更小的步骤。常见的任务分解方法包括向语言模型提供简单提示、针对特定任务量身定制的任务说明，或者融入人类输入以有效引导分解过程。
```

### 将所有步骤整合在一起

![](../../static/img/conversational_retrieval_chain.png)

为方便起见，我们将所有必要步骤整合在一个代码单元中：

```python
import bs4
from langchain.chains import create_history_aware_retriever, create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_chroma import Chroma
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
### 构建检索器 ###
loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs=dict(
        parse_only=bs4.SoupStrainer(
            class_=("post-content", "post-title", "post-header")
        )
    ),
)
docs = loader.load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = text_splitter.split_documents(docs)
vectorstore = Chroma.from_documents(documents=splits, embedding=OpenAIEmbeddings())
retriever = vectorstore.as_retriever()
### 上下文化问题 ###
contextualize_q_system_prompt = (
    "给定一个聊天历史和最新的用户问题，可能涉及聊天历史上下文，"
    "制定一个可以独立理解的问题，不要回答问题，"
    "如果需要，重新制定问题，否则原样返回。"
)
contextualize_q_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", contextualize_q_system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
history_aware_retriever = create_history_aware_retriever(
    llm, retriever, contextualize_q_prompt
)
### 回答问题 ###
system_prompt = (
    "You are an assistant for question-answering tasks. 
"使用以下检索到的上下文片段来回答问题。如果你不知道答案，可以说不知道。最多用三个句子，保持回答简洁。"
"{context}"
)
qa_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
question_answer_chain = create_stuff_documents_chain(llm, qa_prompt)
rag_chain = create_retrieval_chain(history_aware_retriever, question_answer_chain)
### 有状态地管理聊天记录 ###
store = {}
def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]
conversational_rag_chain = RunnableWithMessageHistory(
    rag_chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history",
    output_messages_key="answer",
)
```
```python
conversational_rag_chain.invoke(
    {"input": "什么是任务分解？"},
    config={
        "configurable": {"session_id": "abc123"}
    },  # 在`store`中构建一个键"abc123"。
)["answer"]
```
```output
'任务分解涉及将复杂任务分解为更小、更简单的步骤，使其更易管理。这个过程通过将困难的任务分解为更容易实现的子目标，帮助代理人或模型解决困难任务。任务分解可以通过“思维链”或“思维树”等技术来完成，这些技术指导模型逐步思考或在每一步探索多种推理可能性。'
```
```python
conversational_rag_chain.invoke(
    {"input": "常见的做法有哪些？"},
    config={"configurable": {"session_id": "abc123"}},
)["answer"]
```
```output
"任务分解的常见方法包括使用“思维链”(CoT)或“思维树”等技术，指导模型将复杂任务分解为较小的步骤。这可以通过简单的提示LLMs、任务特定的指令或人类输入来实现，以帮助模型理解和有效地完成任务。任务分解旨在通过利用更多的测试时间计算，并揭示模型的思考过程，提高模型在复杂任务上的性能。"
```

## 代理人 {#agents}

代理人利用LLMs的推理能力在执行过程中做出决策。使用代理人可以让您摆脱对检索过程的某些自由裁量权。尽管它们的行为不如链条那样可预测，但在这种情况下，它们提供了一些优势：

- 代理人直接生成检索工具的输入，而不一定需要我们像上面那样明确地构建上下文；

- 代理人可以执行多个检索步骤以服务于一个查询，或者完全不执行检索步骤（例如，对用户的一般问候做出回应）。

### 检索工具

代理人可以访问“工具”并管理它们的执行。在这种情况下，我们将把我们的检索器转换为LangChain工具，供代理人使用：

```python
from langchain.tools.retriever import create_retriever_tool
tool = create_retriever_tool(
    retriever,
    "blog_post_retriever",
    "搜索并返回《自主代理人》博客文章的摘录。",
)
tools = [tool]
```

### 代理人构造器

现在我们已经定义了工具和LLM，我们可以创建代理人。我们将使用[LangGraph](/docs/concepts/#langgraph)来构建代理人。

目前我们正在使用一个高级接口来构建代理人，但LangGraph的好处在于，这个高级接口是由一个低级、高度可控的API支持的，以便您可以修改代理人逻辑。

```python
from langgraph.prebuilt import chat_agent_executor
agent_executor = chat_agent_executor.create_tool_calling_executor(llm, tools)
```

现在我们可以试一下。请注意，到目前为止它还不是有状态的（我们仍然需要添加内存）：

```python
from langchain_core.messages import HumanMessage
query = "什么是任务分解？"
for s in agent_executor.stream(
    {"messages": [HumanMessage(content=query)]},
):
    print(s)
    print("----")
```
```output
任务分解是一种将复杂任务分解为较小、较简单步骤的技术。这种方法有助于代理在规划和执行任务时更加高效。任务分解的一种常见方法是链式思考（Chain of Thought，CoT）技术，其中模型被指导按步骤思考，将难以完成的任务分解为可管理的步骤。CoT的另一个扩展是思维树（Tree of Thoughts），它通过创建思维步骤的树结构，在每个步骤中探索多种推理可能性。
任务分解可以通过多种方法实现，例如使用简单提示的语言模型、任务特定的指令或人工输入。通过将任务分解为较小的组成部分，代理可以更好地规划和执行任务。
如果您想了解有关任务分解的更详细信息或示例，请随时提问！
```python
config = {"configurable": {"thread_id": "abc123"}}
for s in agent_executor.stream(
    {"messages": [HumanMessage(content="Hi! I'm bob")]}, config=config
):
    print(s)
    print("----")
```
```output

{'agent': {'messages': [AIMessage(content='Hello Bob! How can I assist you today?', response_metadata={'token_usage': {'completion_tokens': 11, 'prompt_tokens': 67, 'total_tokens': 78}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'stop', 'logprobs': None}, id='run-1451e59b-b135-4776-985d-4759338ffee5-0')]}}

----

```
此外，如果我们输入一个需要检索步骤的查询，代理会生成工具的输入：
```python
query = "What is Task Decomposition?"
for s in agent_executor.stream(
    {"messages": [HumanMessage(content=query)]}, config=config
):
    print(s)
    print("----")
```
```output

{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_ab2x4iUPSWDAHS5txL7PspSK', 'function': {'arguments': '{"query":"Task Decomposition"}', 'name': 'blog_post_retriever'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 19, 'prompt_tokens': 91, 'total_tokens': 110}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-f76b5813-b41c-4d0d-9ed2-667b988d885e-0', tool_calls=[{'name': 'blog_post_retriever', 'args': {'query': 'Task Decomposition'}, 'id': 'call_ab2x4iUPSWDAHS5txL7PspSK'}])]}}

----

{'action': {'messages': [ToolMessage(content='Fig. 1. Overview of a LLM-powered autonomous agent system.\nComponent One: Planning#\nA complicated task usually involves many steps. An agent needs to know what they are and plan ahead.\nTask Decomposition#\nChain of thought (CoT; Wei et al. 2022) has become a standard prompting technique for enhancing model performance on complex tasks. The model is instructed to “think step by step” to utilize more test-time computation to decompose hard tasks into smaller and simpler steps. CoT transforms big tasks into multiple manageable tasks and shed lights into an interpretation of the model’s thinking process.\n\nTree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.\nTask decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.\n\n(3) Task execution: Expert models execute on the specific tasks and log results.\nInstruction:\n\nWith the input and the inference results, the AI assistant needs to describe the process and results. The previous stages can be formed as - User Input: {{ User Input }}, Task Planning: {{ Tasks }}, Model Selection: {{ Model Assignment }}, Task Execution: {{ Predictions }}. You must first answer the user\'s request in a straightforward manner. Then describe the task process and show your analysis and model inference results to the user in the first person. If inference results contain a file path, must tell the user the complete file path.\n\nFig. 11. Illustration of how HuggingGPT works. (Image source: Shen et al. 2023)\nThe system comprises of 4 stages:\n(1) Task planning: LLM works as the brain and parses the user requests into multiple tasks. There are four attributes associated with each task: task type, ID, dependencies, and arguments. They use few-shot examples to guide LLM to do task parsing and planning.\nInstruction:', name='blog_post_retriever', id='e0895fa5-5d41-4be0-98db-10a83d42fc2f', tool_call_id='call_ab2x4iUPSWDAHS5txL7PspSK')]}}

----

{'agent': {'messages': [AIMessage(content='Task decomposition is a technique used in complex tasks where the task is broken down into smaller and simpler steps. This approach helps in managing and solving difficult tasks by dividing them into more manageable components. One common method for task decomposition is the Chain of Thought (CoT) technique, which prompts the model to think step by step and decompose hard tasks into smaller steps. Another extension of CoT is the Tree of Thoughts, which explores multiple reasoning possibilities at each step by creating a tree structure of thought steps.\n\nTask decomposition can be achieved through various methods, such as using language models with simple prompting, task-specific instructions, or human inputs. By breaking down tasks into smaller components, agents can better plan and execute complex tasks effectively.\n\nIf you would like more detailed information or examples related to task decomposition, feel free to ask!', response_metadata={'token_usage': {'completion_tokens': 165, 'prompt_tokens': 611, 'total_tokens': 776}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'stop', 'logprobs': None}, id='run-13296566-8577-4d65-982b-a39718988ca3-0')]}}

---- 

```
在这里，代理程序在将我们的查询直接插入工具之前，会去除诸如"what"和"is"等不必要的词语。
同样的原理也使代理程序能够在必要时使用对话的上下文：
```python
query = "What according to the blog post are common ways of doing it? redo the search"
for s in agent_executor.stream(
    {"messages": [HumanMessage(content=query)]}, config=config
):
    print(s)
    print("----")
```
```output

{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_KvoiamnLfGEzMeEMlV3u0TJ7', 'function': {'arguments': '{"query":"common ways of task decomposition"}', 'name': 'blog_post_retriever'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 21, 'prompt_tokens': 930, 'total_tokens': 951}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-dd842071-6dbd-4b68-8657-892eaca58638-0', tool_calls=[{'name': 'blog_post_retriever', 'args': {'query': 'common ways of task decomposition'}, 'id': 'call_KvoiamnLfGEzMeEMlV3u0TJ7'}])]}}

----

{'action': {'messages': [ToolMessage(content='Tree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.\nTask decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.\n\nFig. 1. Overview of a LLM-powered autonomous agent system.\nComponent One: Planning#\nA complicated task usually involves many steps. An agent needs to know what they are and plan ahead.\nTask Decomposition#\nChain of thought (CoT; Wei et al. 2022) has become a standard prompting technique for enhancing model performance on complex tasks. The model is instructed to “think step by step” to utilize more test-time computation to decompose hard tasks into smaller and simpler steps. CoT transforms big tasks into multiple manageable tasks and shed lights into an interpretation of the model’s thinking process.\n\nResources:\n1. Internet access for searches and information gathering.\n2. Long Term memory management.\n3. GPT-3.5 powered Agents for delegation of simple tasks.\n4. File output.\n\nPerformance Evaluation:\n1. Continuously review and analyze your actions to ensure you are performing to the best of your abilities.\n2. Constructively self-criticize your big-picture behavior constantly.\n3. Reflect on past decisions and strategies to refine your approach.\n4. Every command has a cost, so be smart and efficient. Aim to complete tasks in the least number of steps.\n\n(3) Task execution: Expert models execute on the specific tasks and log results.\nInstruction:\n\nWith the input and the inference results, the AI assistant needs to describe the process and results. The previous stages can be formed as - User Input: {{ User Input }}, Task Planning: {{ Tasks }}, Model Selection: {{ Model Assignment }}, Task Execution: {{ Predictions }}. You must first answer the user\'s request in a straightforward manner. Then describe the task process and show your analysis and model inference results to the user in the first person. If inference results contain a file path, must tell the user the complete file path.', name='blog_post_retriever', id='c749bb8e-c8e0-4fa3-bc11-3e2e0651880b', tool_call_id='call_KvoiamnLfGEzMeEMlV3u0TJ7')]}}

----

{'agent': {'messages': [AIMessage(content='According to the blog post, common ways of task decomposition include:\n\n1. Using language models with simple prompting like "Steps for XYZ" or "What are the subgoals for achieving XYZ?"\n2. Utilizing task-specific instructions, for example, using "Write a story outline" for writing a novel.\n3. Involving human inputs in the task decomposition process.\n\nThese methods help in breaking down complex tasks into smaller and more manageable steps, facilitating better planning and execution of the overall task.', response_metadata={'token_usage': {'completion_tokens': 100, 'prompt_tokens': 1475, 'total_tokens': 1575}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'stop', 'logprobs': None}, id='run-98b765b3-f1a6-4c9a-ad0f-2db7950b900f-0')]}}

----

```
请注意，代理程序能够推断我们查询中的"it"指的是"task decomposition"，并因此生成了一个合理的搜索查询，即"common ways of task decomposition"。
### 将其整合在一起
为了方便起见，我们将所有必要步骤汇总到一个代码单元中：
```python
import bs4
```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain.tools.retriever import create_retriever_tool
from langchain_chroma import Chroma
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langgraph.checkpoint.sqlite import SqliteSaver
memory = SqliteSaver.from_conn_string(":memory:")
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
### 构建检索器 ###
loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs=dict(
        parse_only=bs4.SoupStrainer(
            class_=("post-content", "post-title", "post-header")
        )
    ),
)
docs = loader.load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = text_splitter.split_documents(docs)
vectorstore = Chroma.from_documents(documents=splits, embedding=OpenAIEmbeddings())
retriever = vectorstore.as_retriever()
### 构建检索器工具 ###
tool = create_retriever_tool(
    retriever,
    "blog_post_retriever",
    "搜索并返回《Autonomous Agents》博文摘录。"
)
tools = [tool]
agent_executor = create_tool_calling_agent(
    llm, tools, checkpointer=memory
)
```

## 下一步

我们已经介绍了构建基本对话问答应用的步骤：

- 我们使用链条构建了一个可预测生成每个用户输入的搜索查询的应用程序；

- 我们使用代理构建了一个“决定”何时以及如何生成搜索查询的应用程序。

要探索不同类型的检索器和检索策略，请访问[检索器](/docs/how_to#retrievers)部分的操作指南。

要详细了解LangChain对话记忆抽象，请访问[如何添加消息历史（记忆）](/docs/how_to/message_history) LCEL 页面。

要了解更多关于代理的信息，请前往[代理模块](/docs/tutorials/agents)。

