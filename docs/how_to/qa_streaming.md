# 如何从 RAG 应用程序中流式传输结果
本指南介绍了如何从 RAG 应用程序中流式传输结果。它涵盖了从最终输出以及链条的中间步骤（例如，从查询重写）中流式传输令牌的内容。
我们将使用我们在 [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) 博文中构建的 Q&A 应用程序作为示例，该博文由 Lilian Weng 撰写，位于 [RAG 教程](/docs/tutorials/rag) 中。
## 设置
### 依赖项
在本教程中，我们将使用 OpenAI 嵌入和 Chroma 向量存储，但是这里展示的所有内容都适用于任何 [嵌入模型](/docs/concepts#embedding-models)、[向量存储](/docs/concepts#vectorstores) 或 [检索器](/docs/concepts#retrievers)。
我们将使用以下软件包：
```python
%pip install --upgrade --quiet  langchain langchain-community langchainhub langchain-openai langchain-chroma bs4
```
我们需要设置环境变量 `OPENAI_API_KEY`，可以直接设置或从 `.env` 文件中加载，如下所示：
```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
# import dotenv
# dotenv.load_dotenv()
```
### LangSmith
使用 LangChain 构建的应用程序通常包含多个步骤，需要多次调用 LLM。随着这些应用程序变得越来越复杂，能够检查链条或代理内部发生的情况变得至关重要。最好的方法是使用 [LangSmith](https://smith.langchain.com)。
请注意，LangSmith 不是必需的，但它很有帮助。如果您想使用 LangSmith，在上面的链接注册后，请确保设置环境变量以开始记录跟踪：
```python
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```
## RAG 链
首先选择一个 LLM：
```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs customVarName="llm" />
```
这是一个 Q&A 应用程序，我们在 [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) 博文中构建，该博文由 Lilian Weng 撰写，位于 [RAG 教程](/docs/tutorials/rag) 中：
```python
import bs4
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
# 1. 加载、分块和索引博客内容以创建检索器。
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
# 2. 将检索器整合到问答链中。
system_prompt = (
    "You are an assistant for question-answering tasks. "
    "Use the following pieces of retrieved context to answer "
    "the question. If you don't know the answer, say that you "
    "don't know. Use three sentences maximum and keep the "
    "answer concise."
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
## 流式传输最终输出
`create_retrieval_chain` 构建的链条返回一个包含键 `"input"`、`"context"` 和 `"answer"` 的字典。`.stream` 方法默认会按顺序流式传输每个键。
请注意，这里只有 `"answer"` 键以令牌为单位进行流式传输，因为其他组件（如检索）不支持令牌级别的流式传输。
```python
for chunk in rag_chain.stream({"input": "What is Task Decomposition?"}):
    print(chunk)
```
```output
{'input': 'What is Task Decomposition?'}
{'context': [Document(page_content='Fig. 1. Overview of a LLM-powered autonomous agent system.\nComponent One: Planning#\nA complicated task usually involves many steps. An agent needs to know what they are and plan ahead.\nTask Decomposition#\nChain of thought (CoT; Wei et al. 2022) has become a standard prompting technique for enhancing model performance on complex tasks. The model is instructed to “think step by step” to utilize more test-time computation to decompose hard tasks into smaller and simpler steps. CoT transforms big tasks into multiple manageable tasks and shed lights into an interpretation of the model’s thinking process.', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}), Document(page_content='Tree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.\nTask decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.', metadata
```
我们可以在数据流出时自由处理数据块。例如，如果我们只想要流式传输答案标记，我们可以选择具有相应键的数据块：
```python
for chunk in rag_chain.stream({"input": "什么是任务分解?"}):
    if answer_chunk := chunk.get("answer"):
        print(f"{answer_chunk}|", end="")
```
```output
任务|分解|是|一种|技术，用于将|复杂|任务|分解为|更小|更易管理|的步骤|。|这一过程|有助于|代理|或|模型|处理|复杂|任务|，将其|分解为|更简单|的子任务|。通过|任务分解|，模型|可以|有效地|规划|并|执行|朝着|实现|整体目标|的|每一步|。|
```
更简单地说，我们可以使用 [.pick](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.pick) 方法仅选择所需的键：
```python
chain = rag_chain.pick("answer")
for chunk in chain.stream({"input": "什么是任务分解?"}):
    print(f"{chunk}|", end="")
```
```output
|任务|分解|涉及|将|复杂|任务|分解为|更小|更简单|的步骤|，以便|代理|或|模型|更好地|处理|。|这一过程|有助于|高效|规划|和|执行|任务|，将其|分解为|一系列|子目标|或|行动|。|任务|分解|可以通过|Chain of Thought (CoT)|或|Tree of Thoughts|等|技术|实现，这些技术|通过|引导|模型|进行|逐步|思考|过程|，提高|模型|在|复杂|任务|上的|表现|。|
```
## 流式传输中间步骤
假设我们不仅想要流式传输链的最终输出，还想要一些中间步骤。让我们以我们的[对话式 RAG](/docs/tutorials/qa_chat_history)链为例。在将问题传递给检索器之前，我们重新构造用户问题。这个重新构造的问题不会作为最终输出的一部分返回。我们可以修改链以返回新问题，但为了演示目的，我们将保持原样。
```python
from langchain.chains import create_history_aware_retriever
from langchain_core.prompts import MessagesPlaceholder
### 上下文化问题 ###
contextualize_q_system_prompt = (
    "给定一个聊天记录和最新的用户问题，"
    "可能引用聊天记录中的上下文，"
    "制定一个可以理解的独立问题，"
    "而无需聊天记录。如果需要，重新构造问题，"
    "否则原样返回。"
)
contextualize_q_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", contextualize_q_system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
contextualize_q_llm = llm.with_config(tags=["contextualize_q_llm"])
history_aware_retriever = create_history_aware_retriever(
    contextualize_q_llm, retriever, contextualize_q_prompt
)
### 回答问题 ###
system_prompt = (
    "您是一个用于回答问题的助手。"
    "使用以下检索到的上下文片段来回答问题。"
    "如果您不知道答案，请说您不知道。最多使用三句话，"
    "保持答案简洁。"
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
请注意，上面我们使用`.with_config`为用于问题重新表述步骤的 LLM 分配了一个标签。这并非必需，但将使从该特定步骤流式传输输出更加方便。
为了演示，我们将传入一个人工消息历史：
```
人类: 什么是任务分解?
AI: 任务分解涉及将复杂任务分解为更小更简单的步骤。
```
然后我们提出一个后续问题："有哪些常见的方法？"在进入检索步骤之前，我们的`history_aware_retriever`将使用对话的上下文重新表述这个问题，以确保检索是有意义的。
要流式传输中间输出，我们建议使用异步`.astream_events`方法。该方法将从链中的所有“事件”流式传输输出，并且可能会非常冗长。我们可以使用标签、事件类型和其他条件进行过滤，就像我们在这里所做的那样。
下面我们展示了一个典型的`.astream_events`循环，我们传入链输入并发出所需的结果。有关更多详细信息，请参阅[API 参考](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.astream_events)和[流式传输指南](/docs/how_to/streaming)。
```python
first_question = "任务分解是什么意思？"
first_answer = (
    "任务分解是将复杂的任务分解成较小、较简单的步骤的过程。"
)
follow_up_question = "有哪些常见的任务分解方法？"
chat_history = [
    ("人类", first_question),
    ("AI", first_answer),
]
async for event in rag_chain.astream_events(
    {
        "input": follow_up_question,
        "chat_history": chat_history,
    },
    version="v1",
):
    if (
        event["event"] == "on_chat_model_stream"
        and "contextualize_q_llm" in event["tags"]
    ):
        ai_message_chunk = event["data"]["chunk"]
        print(f"{ai_message_chunk.content}|", end="")
```
```output
|有哪些| 典型的| 任务分解| 方法|？||
```
在这里，我们逐个恢复了传递给检索器的查询，给定我们的问题“有哪些常见的任务分解方法？”
如果我们想要获取检索到的文档，我们可以按名称“Retriever”进行过滤：
```python
async for event in rag_chain.astream_events(
    {
        "input": follow_up_question,
        "chat_history": chat_history,
    },
    version="v1",
):
    if event["name"] == "Retriever":
        print(event)
        print()
```
```output
{'event': 'on_retriever_start', 'name': 'Retriever', 'run_id': '6834097c-07fe-42f5-a566-a4780af4d1d0', 'tags': ['seq:step:4', 'Chroma', 'OpenAIEmbeddings'], 'metadata': {}, 'data': {'input': {'query': '有哪些典型的任务分解方法？'}}}
{'event': 'on_retriever_end', 'name': 'Retriever', 'run_id': '6834097c-07fe-42f5-a566-a4780af4d1d0', 'tags': ['seq:step:4', 'Chroma', 'OpenAIEmbeddings'], 'metadata': {}, 'data': {'input': {'query': '有哪些典型的任务分解方法？'}, 'output': {'documents': [Document(page_content='思维树（Yao等人，2023年）通过在每个步骤中探索多种推理可能性来扩展CoT。它首先将问题分解为多个思考步骤，并在每个步骤中生成多个思考，创建树结构。搜索过程可以是BFS（广度优先搜索）或DFS（深度优先搜索），每个状态由分类器（通过提示）或多数投票进行评估。\n任务分解可以通过以下方式进行：（1）使用简单提示的LLM，例如“XYZ的步骤。\\n1。”，“实现XYZ的子目标是什么？”，（2）使用任务特定的说明；例如，写小说时使用“编写故事大纲。”，或（3）使用人类输入。', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}), Document(page_content='图1. LLM驱动的自主代理系统概述。\n组件一：规划#\n复杂的任务通常涉及许多步骤。代理需要知道这些步骤并提前规划。\n任务分解#\n思维链（CoT；Wei等人，2022年）已成为增强模型在复杂任务上性能的标准提示技术。模型被指示“逐步思考”，利用更多的测试时间计算将困难任务分解为较小、较简单的步骤。CoT将大任务转化为多个可管理的任务，并为模型的思考过程提供了解释。', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}), Document(page_content='资源：\n1. 用于搜索和信息收集的互联网访问。\n2. 长期记忆管理。\n3. 用于委派简单任务的GPT-3.5驱动代理。\n4. 文件输出。\n\n性能评估：\n1. 不断回顾和分析您的行动，以确保您发挥出最佳水平。\n2. 不断进行建设性的自我批评，关注整体行为。\n3. 反思过去的决策和策略，以完善您的方法。\n4. 每个命令都有成本，因此要聪明高效。目标是以最少的步骤完成任务。', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}), Document(page_content='图9. MIPS算法的比较，以recall@10为度量。（图片来源：Google Blog，2020年）\n在ann-benchmarks.com上检查更多MIPS算法和性能比较。\n组件三：工具使用#\n工具使用是人类的一个显著和独特特征。我们创造、修改和利用外部对象来完成超出我们物理和认知能力的事情。为LLM配备外部工具可以显著扩展模型的功能。', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'})]}}}
```
有关如何流式传输中间步骤的更多信息，请查看[流式传输指南](/docs/how_to/streaming)。