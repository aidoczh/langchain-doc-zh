---
sidebar_position: 2
---
# 对话式 RAG
在许多问答应用程序中，我们希望允许用户进行来回对话，这意味着应用程序需要某种过去问题和答案的“记忆”，以及一些逻辑来将这些内容融入当前的思考中。
在本指南中，我们专注于**添加逻辑以融入历史消息**。有关聊天历史管理的更多细节请参阅[此处](/docs/how_to/message_history)。
我们将涵盖两种方法：
1. 链，其中我们始终执行检索步骤；
2. 代理，其中我们让一个 LLM 自行决定是否以及如何执行检索步骤（或多个步骤）。
对于外部知识来源，我们将使用同一篇来自 [RAG 教程](/docs/tutorials/rag) 的 Lilian Weng 的[由 LLM 驱动的自主代理](https://lilianweng.github.io/posts/2023-06-23-agent/)博文。
## 设置
### 依赖项
在本教程中，我们将使用 OpenAI 嵌入和 Chroma 向量存储，但这里展示的所有内容都适用于任何[嵌入模型](/docs/concepts#embedding-models)、[向量存储](/docs/concepts#vectorstores)或[检索器](/docs/concepts#retrievers)。
我们将使用以下软件包：
```python
%pip install --upgrade --quiet  langchain langchain-community langchainhub langchain-chroma bs4
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
您使用 LangChain 构建的许多应用程序将包含多个步骤，多次调用 LLM。随着这些应用程序变得越来越复杂，能够检查链或代理内部发生的情况变得至关重要。这样做的最佳方式是使用[LangSmith](https://smith.langchain.com)。
请注意，LangSmith 不是必需的，但很有帮助。如果您想使用 LangSmith，在上面的链接注册后，请确保设置您的环境变量以开始记录跟踪：
```python
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```
## 链 {#chains}
让我们首先回顾一下我们在 Lilian Weng 的[RAG 教程](/docs/tutorials/rag)中构建的 Q&A 应用程序。

import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs customVarName="llm" />

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
```
```python
# 1. 加载、分块和索引博客内容，创建一个检索器。
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
# 2. 将检索器融入问答链。
system_prompt = (
    "您是一个用于问答任务的助手。"
    "使用以下检索到的上下文片段来回答问题。"
    "如果您不知道答案，请说您不知道。"
    "最多使用三句话，保持回答简洁。"
    "\n\n"
    "{context}"
)
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", "{input}"),
    ]
)
question_answer_chain = create_stuff_documents_chain(llm, prompt)
rag_chain = create_retrieval_chain(retriever, question_answer_chain)
```
```python
response = rag_chain.invoke({"input": "什么是任务分解？"})
response["answer"]
```
```output
'任务分解涉及将复杂任务分解为更小、更简单的步骤，以使其更易管理。这一过程可以通过诸如思维链 (CoT) 或思维树等技术来实现，这些技术帮助代理人通过将任务分解为顺序子目标来有效地规划和执行任务。任务分解可以通过使用提示技术、任务特定说明或人类输入来促进，以引导代理人完成任务所需的步骤。'
```
请注意，我们使用了内置的链构造函数 `create_stuff_documents_chain` 和 `create_retrieval_chain`，因此我们解决方案的基本组成部分如下：
1. 检索器（retriever）；
2. 提示（prompt）；
3. 语言模型（LLM）。
这将简化将聊天历史纳入过程的过程。
### 添加聊天历史
我们构建的链直接使用输入查询来检索相关上下文。但在对话环境中，用户的查询可能需要对话上下文才能理解。例如，考虑以下对话：
> 人类：「什么是任务分解？」
>
> AI：「任务分解是将复杂任务分解为更小更简单的步骤，以便于代理或模型更好地处理。」
>
> 人类：「有哪些常见的方法？」
为了回答第二个问题，我们的系统需要理解「它」指的是「任务分解」。
我们需要更新现有应用的两个方面：
1. **提示**：更新提示以支持历史消息作为输入。
2. **上下文化问题**：添加一个子链，将最新的用户问题重新表述为聊天历史的上下文。可以简单地将其视为构建一个新的「具有历史意识」的检索器。之前我们有：
   - `query` -> `retriever`
   现在我们将有：
   - `(query, conversation history)` -> `LLM` -> `rephrased query` -> `retriever`
#### 上下文化问题
首先，我们需要定义一个子链，接受历史消息和最新的用户问题，并在问题引用历史信息时重新表述问题。
我们将使用一个包含 `MessagesPlaceholder` 变量的提示，名称为「chat_history」。这允许我们使用「chat_history」输入键将消息列表传递给提示，这些消息将在系统消息之后、包含最新问题的人类消息之前插入。
请注意，我们利用了一个辅助函数 [create_history_aware_retriever](https://api.python.langchain.com/en/latest/chains/langchain.chains.history_aware_retriever.create_history_aware_retriever.html) 来管理 `chat_history` 为空的情况，并按顺序应用 `prompt | llm | StrOutputParser() | retriever`。
`create_history_aware_retriever` 构建了一个链，接受 `input` 和 `chat_history` 作为输入键，并具有与检索器相同的输出模式。
```python
from langchain.chains import create_history_aware_retriever
from langchain_core.prompts import MessagesPlaceholder
contextualize_q_system_prompt = (
    "给定聊天历史和最新的用户问题，"
    "该问题可能引用聊天历史中的上下文，"
    "重新构造一个可以在没有聊天历史的情况下理解的独立问题。"
    "如果需要，不要回答问题，只需重新构造问题并返回。"
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
```
这个链在我们的检索器之前添加了输入查询的重新表述，以便检索过程中包含对话的上下文。
现在我们可以构建完整的问答链。只需将检索器更新为我们的新 `history_aware_retriever` 即可。
同样，我们将使用 [create_stuff_documents_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.combine_documents.stuff.create_stuff_documents_chain.html) 来生成一个 `question_answer_chain`，其输入键为 `context`、`chat_history` 和 `input`，它接受检索到的上下文以及对话历史和查询来生成答案。
我们使用 [create_retrieval_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.retrieval.create_retrieval_chain.html) 构建最终的 `rag_chain`。该链按顺序应用 `history_aware_retriever` 和 `question_answer_chain`，保留中间输出，如检索到的上下文，以方便使用。它的输入键为 `input` 和 `chat_history`，并在输出中包含 `input`、`chat_history`、`context` 和 `answer`。
```python
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
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
让我们试一下。下面我们提出一个问题和一个需要上下文化才能返回合理回答的后续问题。由于我们的链包括了一个「chat_history」输入，调用者需要管理聊天历史。我们可以通过将输入和输出消息附加到一个列表来实现这一点：
```python
from langchain_core.messages import AIMessage, HumanMessage
chat_history = []
question = "什么是任务分解？"
ai_msg_1 = rag_chain.invoke({"input": question, "chat_history": chat_history})
chat_history.extend(
    [
        HumanMessage(content=question),
        AIMessage(content=ai_msg_1["answer"]),
    ]
)
second_question = "常见的做法有哪些？"
ai_msg_2 = rag_chain.invoke({"input": second_question, "chat_history": chat_history})
print(ai_msg_2["answer"])
```
```output
任务分解可以通过几种常见的方式来完成，例如使用语言模型（LLM）进行简单提示，比如“XYZ的步骤”，或者要求实现特定任务的子目标。还可以提供任务特定的说明，比如请求写小说的故事大纲。此外，可以利用人类输入有效地将任务分解为较小的组件。
```
:::tip
查看 [LangSmith 追踪](https://smith.langchain.com/public/243301e4-4cc5-4e52-a6e7-8cfe9208398d/r) 
:::
#### 对话历史的有状态管理
在这里，我们已经介绍了如何添加应用程序逻辑以合并历史输出，但我们仍在手动更新对话历史并将其插入到每个输入中。在真正的问答应用程序中，我们希望有一种持久化对话历史的方式，并且有一种自动插入和更新它的方式。
为此，我们可以使用：
- [BaseChatMessageHistory](https://api.python.langchain.com/en/latest/langchain_api_reference.html#module-langchain.memory): 存储对话历史。
- [RunnableWithMessageHistory](/docs/how_to/message_history): LCEL 链和 `BaseChatMessageHistory` 的包装器，负责将对话历史注入输入并在每次调用后更新它。
要详细了解如何将这些类结合在一起创建有状态的对话链，请转到 [如何添加消息历史（内存）](/docs/how_to/message_history) LCEL 页面。
下面，我们实现了第二种选项的一个简单示例，其中对话历史存储在一个简单的字典中。LangChain 通过与 [Redis](/docs/integrations/memory/redis_chat_message_history/) 和其他技术的内存集成来提供更强大的持久性。
`RunnableWithMessageHistory` 的实例会为您管理对话历史。它们接受一个带有键（默认为 `"session_id"`）的配置，该键指定要获取和预置到输入中的对话历史，并将输出附加到相同的对话历史。以下是一个示例：
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
    },  # 在 `store` 中构建一个键为 "abc123" 的键。
)["answer"]
```
```output
'任务分解涉及将复杂任务分解为较小且更简单的步骤，以使其对代理或模型更易管理。这个过程有助于引导代理通过实现整体任务所需的各种子目标。可以使用不同的技术，如思维链和思维树，来将任务分解为可管理的组件。'
```
```python
conversational_rag_chain.invoke(
    {"input": "常见的做法有哪些?"},
    config={"configurable": {"session_id": "abc123"}},
)["answer"]
```
```output
'任务分解可以通过各种方法实现，例如使用提示技术如“XYZ的步骤”来引导模型完成子目标，为特定任务提供任务特定说明如“写一个故事大纲”，或者整合人类输入来分解复杂任务。这些方法有助于将大任务分解为更小、更易管理的组件，以便更好地理解和执行。'
```
对话历史可以在 `store` 字典中检查：
```python
for message in store["abc123"].messages:
    if isinstance(message, AIMessage):
        prefix = "AI"
    else:
        prefix = "User"
    print(f"{prefix}: {message.content}\n")
```
```output
User: 什么是任务分解?
AI: 任务分解涉及将复杂任务分解为较小且更简单的步骤，以使其对代理或模型更易管理。这个过程有助于引导代理通过实现整体任务所需的各种子目标。可以使用不同的技术，如思维链和思维树，来将任务分解为可管理的组件。
User: 常见的做法有哪些?
AI: 任务分解可以通过各种方法实现，例如使用提示技术如“XYZ的步骤”来引导模型完成子目标，为特定任务提供任务特定说明如“写一个故事大纲”，或者整合人类输入来分解复杂任务。这些方法有助于将大任务分解为更小、更易管理的组件，以便更好地理解和执行。
```
### 将一切联系在一起
![](../../static/img/conversational_retrieval_chain.png)
为了方便起见，我们将所有必要的步骤汇总在一个单独的代码单元中：
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
    "给定一个聊天历史记录和最新的用户问题，"
    "可能会涉及聊天历史记录中的上下文，"
    "制定一个独立的问题，可以在没有聊天历史记录的情况下理解。"
    "如果需要，重新构造问题，否则原样返回。"
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
    "您是一个用于回答问题的助手。"
    "使用检索到的上下文来回答问题。"
    "如果不知道答案，请说不知道。"
    "最多使用三句话，保持回答简洁。"
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
### 有状态地管理聊天历史记录 ###
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
'任务分解涉及将复杂任务分解为更小更简单的步骤，以使其更易管理。这个过程帮助代理人或模型通过将任务分解为更容易实现的子目标来解决困难任务。任务分解可以通过“思维链”或“思维树”等技术来完成，这些技术指导模型逐步思考或在每个步骤探索多种推理可能性。'
```
```python
conversational_rag_chain.invoke(
    {"input": "常见的做法有哪些？"},
    config={"configurable": {"session_id": "abc123"}},
)["answer"]
```
```output
"任务分解的常见方法包括使用“思维链”（CoT）或“思维树”等技术来指导模型将复杂任务分解为较小的步骤。这可以通过简单提示LLMs、任务特定指令或人类输入来实现，以帮助模型理解和有效地导航任务。任务分解旨在通过利用更多的测试时间计算并揭示模型的思维过程，提高模型在复杂任务上的性能。"
```
## 代理人 {#agents}
代理人利用LLMs的推理能力在执行过程中做出决策。使用代理人可以让您在检索过程中分担一些自主权。尽管它们的行为不如链条那样可预测，但在这种情况下，它们提供了一些优势：
- 代理人直接生成检索器的输入，无需像上面那样明确构建上下文化；
- 代理人可以执行多个检索步骤以服务于查询，或完全不执行检索步骤（例如，响应用户的通用问候）。
### 检索工具
代理人可以访问“工具”并管理其执行。在这种情况下，我们将把我们的检索器转换为LangChain工具，供代理人使用：
```python
from langchain.tools.retriever import create_retriever_tool
tool = create_retriever_tool(
    retriever,
    "blog_post_retriever",
    "搜索并返回自主代理博客文章摘录。",
)
tools = [tool]
```
工具是 LangChain [Runnables](/docs/concepts#langchain-expression-language)，并实现了通常的接口：
```python
tool.invoke("任务分解")
```
```output
'思维树（Yao 等人，2023）通过探索每一步的多种推理可能性扩展了 CoT。它首先将问题分解为多个思考步骤，并在每一步生成多个思考，从而创建了一种树状结构。搜索过程可以是 BFS（广度优先搜索）或 DFS（深度优先搜索），每个状态由分类器（通过提示）或多数投票评估。\n任务分解可以通过以下方式进行：（1）使用简单提示的 LLM，如“XYZ 的步骤。\\n1。”，“实现 XYZ 的子目标是什么？”，（2）使用特定于任务的说明；例如，为写小说而写“写故事大纲。”，或（3）使用人类输入。\n\n图 1. LLM 驱动的自主代理系统概览。\n组件一：规划#\n复杂的任务通常涉及许多步骤。代理需要知道这些步骤并提前规划。\n任务分解#\n思维链（CoT；Wei 等人，2022）已成为增强模型在复杂任务上性能的标准提示技术。模型被指示“逐步思考”，利用更多的测试时间计算将困难任务分解为更小更简单的步骤。CoT 将大任务转化为多个可管理的任务，并为模型的思考过程提供了解释。\n\n（3）任务执行：专家模型执行特定任务并记录结果。\n说明：\n\n通过输入和推理结果，AI 助手需要描述过程和结果。前几个阶段可以形成为 - 用户输入：{{ 用户输入 }}, 任务规划：{{ 任务 }}, 模型选择：{{ 模型分配 }}, 任务执行：{{ 预测 }}。您必须先直接回答用户的请求。然后描述任务过程，并以第一人称向用户展示您的分析和模型推理结果。如果推理结果包含文件路径，必须告诉用户完整的文件路径。\n\n图 11. HuggingGPT 工作原理示意图。（图片来源：Shen 等人，2023）\n该系统包括 4 个阶段：\n（1）任务规划：LLM 作为大脑，将用户请求解析为多个任务。每个任务都有四个属性：任务类型、ID、依赖关系和参数。他们使用少量示例来指导 LLM 进行任务解析和规划。\n说明：'
```
### 代理构造函数
现在我们已经定义了工具和 LLM，我们可以创建代理。我们将使用 [LangGraph](/docs/concepts/#langgraph) 来构建代理。
目前我们正在使用高级接口来构建代理，但 LangGraph 的好处在于，这种高级接口由低级、高度可控的 API 支持，以防您想修改代理逻辑。
```python
from langgraph.prebuilt import chat_agent_executor
agent_executor = chat_agent_executor.create_tool_calling_executor(llm, tools)
```
现在我们可以尝试一下。请注意，到目前为止它还不是有状态的（我们仍然需要添加内存）。
```python
query = "什么是任务分解？"
for s in agent_executor.stream(
    {"messages": [HumanMessage(content=query)]},
):
    print(s)
    print("----")
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_demTlnha4vYA1IH6CByYupBQ', 'function': {'arguments': '{"query":"任务分解"}', 'name': 'blog_post_retriever'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 19, 'prompt_tokens': 68, 'total_tokens': 87}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-d1c3f3da-be18-46a5-b3a8-4621ba1f7f2a-0', tool_calls=[{'name': 'blog_post_retriever', 'args': {'query': '任务分解'}, 'id': 'call_demTlnha4vYA1IH6CByYupBQ'}])]}}
----
{'action': {'messages': [ToolMessage(content='图 1. LLM 驱动的自主代理系统概览。\n组件一：规划#\n复杂的任务通常涉及许多步骤。代理需要知道这些步骤并提前规划。\n任务分解#\n思维链（CoT；Wei 等人，2022）已成为增强模型在复杂任务上性能的标准提示技术。模型被指示“逐步思考”，利用更多的测试时间计算将困难任务分解为更小更简单的步骤。CoT 将大任务转化为多个可管理的任务，并为模型的思考过程提供了解释。\n\nTree of Thoughts（Yao 等人，2023）通过探索每一步的多种推理可能性扩展了 CoT，并创建了任务的树状结构。\n\n任务分解可以通过各种方法实现，例如使用简单提示进行语言模型、特定任务说明或人类输入。通过将任务分解为较小的组件，代理可以更有效地规划和执行任务。\n\n总之，任务分解是自主代理处理复杂任务的有价值策略，将其分解为较小、更易管理的步骤。', name='blog_post_retriever', id='e83e4002-33d2-46ff-82f4-fddb3035fb6a', tool_call_id='call_demTlnha4vYA1IH6CByYupBQ')]}}
----
{'agent': {'messages': [AIMessage(content='任务分解是自主代理系统中用于将复杂任务分解为较小、更简单步骤的技术。这种方法有助于代理更好地理解并规划完成任务所涉及的各个步骤。任务分解的一种常见方法是思维链（CoT）技术，其中模型被提示“逐步思考”，将困难任务分解为可管理的步骤。另一种方法称为思维树，通过探索每一步的多种推理可能性扩展了 CoT，并创建了任务的树状结构。\n\n任务分解可以通过各种方法实现，例如使用简单提示进行语言模型、特定任务说明或人类输入。通过将任务分解为较小组件，代理可以更有效地规划和执行任务。\n\n总的来说，任务分解是自主代理处理复杂任务的有价值策略，将其分解为较小、更易管理的步骤。', response_metadata={'token_usage': {'completion_tokens': 177, 'prompt_tokens': 588, 'total_tokens': 765}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'stop', 'logprobs': None}, id='run-808f32b9-ae61-4f31-a55a-f30643594282-0')]}}
---- 
```
LangGraph 集成了内置的持久性，因此我们不需要使用 ChatMessageHistory！相反，我们可以直接向 LangGraph 代理传递一个检查点器
```python
from langgraph.checkpoint.sqlite import SqliteSaver
memory = SqliteSaver.from_conn_string(":memory:")
agent_executor = chat_agent_executor.create_tool_calling_executor(
    llm, tools, checkpointer=memory
)
```
这就是构建会话式 RAG 代理所需的全部内容。
让我们观察其行为。请注意，如果我们输入一个不需要检索步骤的查询，代理就不会执行检索：
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
{'action': {'messages': [ToolMessage(content='Fig. 1. LLM 驱动的自主代理系统概览。\n组件一：规划#\n复杂任务通常涉及许多步骤。代理需要知道这些步骤并提前规划。\n任务分解#\n思维链（CoT；Wei 等人，2022）已成为增强模型在复杂任务上性能的标准提示技术。模型被指示“逐步思考”，利用更多的测试时间计算将困难任务分解为更小、更简单的步骤。CoT将大任务转化为多个可管理的任务，并揭示模型思考过程的解释。\n\n思维树（Yao 等人，2023）通过探索每个步骤的多个推理可能性扩展了 CoT。它首先将问题分解为多个思考步骤，并在每个步骤生成多个思考，形成树状结构。搜索过程可以是 BFS（广度优先搜索）或 DFS（深度优先搜索），每个状态由分类器（通过提示）或多数投票评估。\n任务分解可以通过以下方式进行：（1）LLM 简单提示，如“XYZ 的步骤。\\n1.”，“实现 XYZ 的子目标是什么？”，（2）使用特定于任务的说明；例如，为写小说而写“撰写故事大纲。”，或（3）通过人类输入。\n\n（3）任务执行：专家模型在特定任务上执行并记录结果。\n说明：\n\n通过输入和推理结果，AI 助手需要简明地描述过程和结果。前几个阶段可以形成为 - 用户输入：{{ 用户输入 }}, 任务规划：{{ 任务 }}, 模型选择：{{ 模型分配 }}, 任务执行：{{ 预测 }}。您必须先直接回答用户的请求。然后描述任务过程，并以第一人称向用户展示分析和模型推理结果。如果推理结果包含文件路径，必须告知用户完整的文件路径。\n\n图 11. HuggingGPT 的工作原理示意图。（图片来源：Shen 等人，2023）\n系统包括 4 个阶段：\n（1）任务规划：LLM 作为大脑，将用户请求解析为多个任务。每个任务关联四个属性：任务类型、ID、依赖关系和参数。他们使用少量示例来指导 LLM 进行任务解析和规划。\n说明:', name='blog_post_retriever', id='e0895fa5-5d41-4be0-98db-10a83d42fc2f', tool_call_id='call_ab2x4iUPSWDAHS5txL7PspSK')]}}
---- 
任务分解是一种在复杂任务中使用的技术，其中任务被分解为更小、更简单的步骤。这种方法有助于通过将困难任务分解为更易管理的组件来管理和解决困难任务。任务分解的一种常见方法是“思维链”（Chain of Thought，CoT）技术，它促使模型逐步思考，并将艰难的任务分解为较小的步骤。CoT的另一个扩展是“思维树”，它通过创建思维步骤的树状结构，在每个步骤探索多种推理可能性。
任务分解可以通过各种方法实现，例如使用带有简单提示、任务特定说明或人类输入的语言模型。通过将任务分解为较小的组件，代理可以更好地规划和有效执行复杂任务。
如果您想了解有关任务分解的更详细信息或示例，请随时提问！
{'action': {'messages': [ToolMessage(content='《思维之树》（Yao等，2023年）通过探索每一步骤的多种推理可能性扩展了CoT。它首先将问题分解为多个思维步骤，并在每个步骤生成多个思维，从而创建了一种树形结构。搜索过程可以是BFS（广度优先搜索）或DFS（深度优先搜索），每个状态由分类器（通过提示）或多数投票评估。\n任务分解可以通过以下方式进行：（1）LLM使用简单提示，如“XYZ的步骤。\\n1.”，“实现XYZ的子目标是什么？”；（2）使用特定任务说明，例如写小说时使用“写故事大纲。”；（3）使用人类输入。\n\n图1. LLM驱动的自主代理系统概述。\n组件一：规划#\n复杂任务通常涉及许多步骤。代理需要知道这些步骤并提前规划。\n任务分解#\n思维链（CoT；Wei等，2022年）已成为增强模型在复杂任务上性能的标准提示技术。模型被指示“逐步思考”，利用更多的测试时间计算将困难任务分解为更小更简单的步骤。CoT将大任务转化为多个可管理的任务，并揭示了模型思考过程的解释。\n\n资源：\n1. 用于搜索和信息收集的互联网访问。\n2. 长期记忆管理。\n3. 用于委派简单任务的GPT-3.5驱动代理。\n4. 文件输出。\n\n性能评估：\n1. 持续审查和分析您的行动，以确保您发挥最佳能力。\n2. 不断对自己的整体行为进行建设性的自我批评。\n3. 反思过去的决策和策略，以完善您的方法。\n4. 每个命令都有成本，因此要聪明高效。目标是以最少的步骤完成任务。\n\n（3）任务执行：专家模型执行特定任务并记录结果。\n指令：\n\n根据输入和推理结果，AI助手需要描述过程和结果。前几个阶段可以形成为-用户输入：{{用户输入}}，任务规划：{{任务}}，模型选择：{{模型分配}}，任务执行：{{预测}}。您必须以直接的方式回答用户的请求。然后描述任务过程，并以第一人称向用户展示您的分析和模型推理结果。如果推理结果包含文件路径，必须告诉用户完整的文件路径。', name='blog_post_retriever', id='c749bb8e-c8e0-4fa3-bc11-3e2e0651880b', tool_call_id='call_KvoiamnLfGEzMeEMlV3u0TJ7')]}}
----
{'agent': {'messages': [AIMessage(content='根据博客文章，任务分解的常见方法包括：\n\n1. 使用语言模型进行简单提示，如“XYZ的步骤”或“实现XYZ的子目标是什么？”\n2. 利用特定任务说明，例如在写小说时使用“写故事大纲”。\n3. 在任务分解过程中涉及人类输入。\n\n这些方法有助于将复杂任务分解为更小更易管理的步骤，促进整体任务的更好规划和执行。', response_metadata={'token_usage': {'completion_tokens': 100, 'prompt_tokens': 1475, 'total_tokens': 1575}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'stop', 'logprobs': None}, id='run-98b765b3-f1a6-4c9a-ad0f-2db7950b900f-0')]}}
----
```
请注意，代理能够推断我们查询中的“it”指的是“任务分解”，并生成了一个合理的搜索查询结果--在这种情况下是“任务分解的常见方法”。
### 将其联系在一起
为了方便起见，我们将所有必要步骤在一个代码单元格中绑定在一起：
```python
import bs4
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
### 构建检索工具 ###
tool = create_retriever_tool(
    retriever,
    "blog_post_retriever",
    "搜索并返回《自主代理》博客文章摘录。",
)
tools = [tool]
agent_executor = chat_agent_executor.create_tool_calling_executor(
    llm, tools, checkpointer=memory
)
```
## 下一步
我们已经介绍了构建基本对话问答应用的步骤：
- 我们使用链式结构构建了一个可预测的应用程序，为每个用户输入生成搜索查询；
- 我们使用代理构建了一个“决定”何时以及如何生成搜索查询的应用程序。
要探索不同类型的检索器和检索策略，请访问[检索器](/docs/how_to/#retrievers)部分的操作指南。
要详细了解LangChain对话记忆抽象，请访问[如何添加消息历史（记忆）](/docs/how_to/message_history) LCEL 页面。
要了解更多关于代理的信息，请前往[代理模块](/docs/tutorials/agents)。