# 如何使你的 RAG 应用返回来源

在问答应用中，向用户展示生成答案所使用的来源非常重要。最简单的方法是让链条返回每一次生成中检索到的文档。

我们将以我们在 [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) 博文中构建的 Q&A 应用为基础，该博文由 Lilian Weng 撰写，位于 [RAG 教程](/docs/tutorials/rag) 中。

我们将介绍两种方法：

1. 使用内置的 [create_retrieval_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.retrieval.create_retrieval_chain.html)，该方法默认返回来源；

2. 使用简单的 [LCEL](/docs/concepts#langchain-expression-language) 实现，以展示工作原理。

## 设置

### 依赖项

在本教程中，我们将使用 OpenAI 的嵌入和 Chroma 向量存储，但这里展示的所有内容都适用于任何 [嵌入模型](/docs/concepts#embedding-models)、[向量存储](/docs/concepts#vectorstores) 或 [检索器](/docs/concepts#retrievers)。

我们将使用以下包：

```python
%pip install --upgrade --quiet  langchain langchain-community langchainhub langchain-openai langchain-chroma bs4
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

使用 LangChain 构建的应用程序通常包含多个步骤，需要多次调用 LLM。随着应用程序越来越复杂，能够检查链条或代理内部发生的情况变得至关重要。最好的方法是使用 [LangSmith](https://smith.langchain.com)。

请注意，LangSmith 不是必需的，但它很有帮助。如果您想使用 LangSmith，请在上面的链接注册后，确保设置环境变量以开始记录跟踪信息：

```python
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 使用 `create_retrieval_chain`

首先选择一个 LLM：

```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs customVarName="llm" />
```

这是我们在 [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) 博文中构建的带有来源的 Q&A 应用：

```python
import bs4
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
# 1. 加载、分块和索引博文内容以创建检索器。
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
# 2. 将检索器整合到问答链条中。
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

```python
result = rag_chain.invoke({"input": "What is Task Decomposition?"})
```

请注意，`result` 是一个带有键 `"input"`、`"context"` 和 `"answer"` 的字典：

```python
result
```

```output
{'input': 'What is Task Decomposition?',
 'context': [Document(page_content='Fig. 1. Overview of a LLM-powered autonomous agent system.\nComponent One: Planning#\nA complicated task usually involves many steps. An agent needs to know what they are and plan ahead.\nTask Decomposition#\nChain of thought (CoT; Wei et al. 2022) has become a standard prompting technique for enhancing model performance on complex tasks. The model is instructed to “think step by step” to utilize more test-time computation to decompose hard tasks into smaller and simpler steps. CoT transforms big tasks into multiple manageable tasks and shed lights into an interpretation of the model’s thinking process.', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}),
  Document(page_content='Tree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.\nTask decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}),
  Document(page_content='Resources:\n1. Internet access for searches and information gathering.\n2. Long Term memory management.\n3. GPT-3.5 powered Agents for delegation of simple tasks.\n4. File output.\n\nPerformance Evaluation:\n1. Continuously review and analyze your actions to ensure you are performing to the best of your abilities.\n2. Constructively self-criticize your big-picture behavior constantly.\n3. Reflect on past decisions and strategies to refine your approach.\n4. Every command has a cost, so be smart and efficient. Aim to complete tasks in the least number of steps.', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}),
  Document(page_content="(3) Task execution: Expert models execute on the specific tasks and log results.\nInstruction:\n\nWith the input and the inference results, the AI assistant needs to describe the process and results. The previous stages can be formed as - User Input: {{ User Input }}, Task Planning: {{ Tasks }}, Model Selection: {{ Model Assignment }}, Task Execution: {{ Predictions }}. You must first answer the user's request in a straightforward manner. Then describe the task process and show your analysis and model inference results to the user in the first person. If inference results contain a file path, must tell the user the complete file path.", metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'})],
 'answer': 'Task decomposition involves breaking down a complex task into smaller and simpler steps. This process helps agents or models handle challenging tasks by dividing them into more manageable subtasks. Techniques like Chain of Thought and Tree of Thoughts are used to decompose tasks into multiple steps for better problem-solving.'}
```

在这里，“context”包含了LLM在生成“answer”中回答时使用的来源。

## 自定义LCEL实现

下面我们构建一个类似于`create_retrieval_chain`构建的链条。它通过构建一个字典来运作：

1. 从包含输入查询的字典开始，将检索到的文档添加到“context”键中；

2. 将查询和上下文都输入到一个RAG链中，并将结果添加到字典中。

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)
rag_chain_from_docs = (
    RunnablePassthrough.assign(context=(lambda x: format_docs(x["context"])))
    | prompt
    | llm
    | StrOutputParser()
)
retrieve_docs = (lambda x: x["input"]) | retriever
chain = RunnablePassthrough.assign(context=retrieve_docs).assign(
    answer=rag_chain_from_docs
)
chain.invoke({"input": "What is Task Decomposition"})
```

```output
{'input': 'What is Task Decomposition',
 'context': [Document(page_content='Fig. 1. Overview of a LLM-powered autonomous agent system.\nComponent One: Planning#\nA complicated task usually involves many steps. An agent needs to know what they are and plan ahead.\nTask Decomposition#\nChain of thought (CoT; Wei et al. 2022) has become a standard prompting technique for enhancing model performance on complex tasks. The model is instructed to “think step by step” to utilize more test-time computation to decompose hard tasks into smaller and simpler steps. CoT transforms big tasks into multiple manageable tasks and shed lights into an interpretation of the model’s thinking process.', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}),
  Document(page_content='Tree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.\nTask decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}),
  Document(page_content='The AI assistant can parse user input to several tasks: [{"task": task, "id", task_id, "dep": dependency_task_ids, "args": {"text": text, "image": URL, "audio": URL, "video": URL}}]. The "dep" field denotes the id of the previous task which generates a new resource that the current task relies on. A special tag "-task_id" refers to the generated text image, audio and video in the dependency task with id as task_id. The task MUST be selected from the following options: {{ Available Task List }}. There is a logical relationship between tasks, please note their order. If the user input can\'t be parsed, you need to reply empty JSON. Here are several cases for your reference: {{ Demonstrations }}. The chat history is recorded as {{ Chat History }}. From this chat history, you can find the path of the user-mentioned resources for your task planning.', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}),
  Document(page_content='Fig. 11. Illustration of how HuggingGPT works. (Image source: Shen et al. 2023)\nThe system comprises of 4 stages:\n(1) Task planning: LLM works as the brain and parses the user requests into multiple tasks. There are four attributes associated with each task: task type, ID, dependencies, and arguments. They use few-shot examples to guide LLM to do task parsing and planning.\nInstruction:', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'})],
 'answer': 'Task decomposition involves breaking down complex tasks into smaller and simpler steps to make them more manageable for autonomous agents or models. This process can be achieved by techniques like Chain of Thought (CoT) or Tree of Thoughts, which guide the model to think step by step or explore multiple reasoning possibilities at each step. Task decomposition can be done through simple prompting with language models, task-specific instructions, or human inputs.'}
```

:::tip

查看[LangSmith trace](https://smith.langchain.com/public/0cb42685-e29e-4280-a503-bef2014d7ba2/r)

:::

```