# 如何为聊天机器人添加检索功能
检索是聊天机器人常用的一种技术，用于在聊天模型的训练数据之外增加数据以增强其响应。本节将介绍如何在聊天机器人的上下文中实现检索，但值得注意的是，检索是一个非常微妙和深入的话题 - 我们鼓励您探索[文档的其他部分](/docs/how_to#qa-with-rag)，这些部分会更深入地介绍这个话题！
## 设置
您需要安装一些软件包，并将您的 OpenAI API 密钥设置为名为 `OPENAI_API_KEY` 的环境变量：
```python
%pip install -qU langchain langchain-openai langchain-chroma beautifulsoup4
# 设置环境变量 OPENAI_API_KEY 或从 .env 文件中加载：
import dotenv
dotenv.load_dotenv()
```
```output
警告：您正在使用 pip 版本 22.0.4；然而，版本 23.3.2 可用。
您应该考虑通过 '/Users/jacoblee/.pyenv/versions/3.10.5/bin/python -m pip install --upgrade pip' 命令进行升级。
注意：您可能需要重新启动内核以使用更新后的软件包。
```
```output
True
```
让我们也设置一个聊天模型，我们将在下面的示例中使用。
```python
from langchain_openai import ChatOpenAI
chat = ChatOpenAI(model="gpt-3.5-turbo-1106", temperature=0.2)
```
## 创建检索器
我们将使用[LangSmith文档](https://docs.smith.langchain.com/overview)作为源材料，并将内容存储在一个向量存储中以供以后检索。请注意，本示例将略过一些关于解析和存储数据源的具体细节 - 您可以在[这里](/docs/how_to#qa-with-rag)看到更多关于创建检索系统的深入文档。
让我们使用一个文档加载器从文档中提取文本：
```python
from langchain_community.document_loaders import WebBaseLoader
loader = WebBaseLoader("https://docs.smith.langchain.com/overview")
data = loader.load()
```
接下来，我们将其分成较小的块，以便 LLM 的上下文窗口可以处理，并将其存储在一个向量数据库中：
```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
all_splits = text_splitter.split_documents(data)
```
然后，我们将这些块嵌入并存储在一个向量数据库中：
```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
vectorstore = Chroma.from_documents(documents=all_splits, embedding=OpenAIEmbeddings())
```
最后，让我们从我们初始化的向量存储中创建一个检索器：
```python
# k 是要检索的块数
retriever = vectorstore.as_retriever(k=4)
docs = retriever.invoke("Can LangSmith help test my LLM applications?")
docs
```
```output
[Document(page_content='Skip to main content🦜️🛠️ LangSmith DocsPython DocsJS/TS DocsSearchGo to AppLangSmithOverviewTracingTesting & EvaluationOrganizationsHubLangSmith CookbookOverviewOn this pageLangSmith Overview and User GuideBuilding reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.Over the past two months, we at LangChain', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
 Document(page_content='LangSmith Overview and User Guide | 🦜️🛠️ LangSmith', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
 Document(page_content='You can also quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs.Monitoring\u200bAfter all this, your app might finally ready to go in production. LangSmith can also be used to monitor your application in much the same way that you used for debugging. You can log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise. Each run can also be', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
 Document(page_content="does that affect the output?\u200bSo you notice a bad output, and you go into LangSmith to see what's going on. You find the faulty LLM call and are now looking at the exact input. You want to try changing a word or a phrase to see what happens -- what do you do?We constantly ran into this issue. Initially, we copied the prompt to a playground of sorts. But this got annoying, so we built a playground of our own! When examining an LLM call, you can click the Open in Playground button to access this", metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'})]
```
我们可以看到，调用上面的检索器会导致 LangSmith 文档的某些部分，其中包含我们的聊天机器人在回答问题时可以使用的测试信息。现在我们有了一个可以从 LangSmith 文档中返回相关数据的检索器！
## 文档链
现在我们有了一个可以返回 LangChain 文档的检索器，让我们创建一个可以使用它们作为上下文来回答问题的链。我们将使用 `create_stuff_documents_chain` 辅助函数来将所有输入文档“填充”到提示中。它还将处理将文档格式化为字符串的工作。
除了聊天模型，该函数还期望有一个具有 `context` 变量的提示，以及一个名为 `messages` 的聊天历史消息占位符。我们将创建一个适当的提示并将其传递如下：
```python
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
SYSTEM_TEMPLATE = """
回答用户的问题基于以下上下文。
如果上下文中没有与问题相关的信息，请不要凭空捏造，只需说“我不知道”：
<context>
{context}
</context>
"""
question_answering_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            SYSTEM_TEMPLATE,
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)
document_chain = create_stuff_documents_chain(chat, question_answering_prompt)
```
我们可以单独调用这个 `document_chain` 来回答问题。让我们使用上面检索到的文档和同样的问题 `LangSmith 如何帮助测试？`：
```python
from langchain_core.messages import HumanMessage
document_chain.invoke(
    {
        "context": docs,
        "messages": [
            HumanMessage(content="LangSmith 能帮助测试我的 LLM 应用吗？")
        ],
    }
)
```
```output
'是的，LangSmith 可以帮助测试和评估您的 LLM 应用。它简化了初始设置，您可以使用它来监视应用程序，记录所有跟踪，可视化延迟和令牌使用统计信息，并在出现特定问题时进行故障排除。'
```
看起来不错！为了比较，我们可以尝试不使用上下文文档并比较结果：
```python
document_chain.invoke(
    {
        "context": [],
        "messages": [
            HumanMessage(content="LangSmith 能帮助测试我的 LLM 应用吗？")
        ],
    }
)
```
```output
"我不清楚 LangSmith 对测试 LLM 应用的具体能力。最好直接联系 LangSmith 询问他们的服务以及他们如何协助测试您的 LLM 应用。"
```
我们可以看到 LLM 没有返回任何结果。
## 检索链
让我们将这个文档链与检索器结合起来。以下是这种方式的一种可能性：
```python
from typing import Dict
from langchain_core.runnables import RunnablePassthrough
def parse_retriever_input(params: Dict):
    return params["messages"][-1].content
retrieval_chain = RunnablePassthrough.assign(
    context=parse_retriever_input | retriever,
).assign(
    answer=document_chain,
)
```
给定一个输入消息列表，我们提取列表中最后一条消息的内容，并将其传递给检索器以获取一些文档。然后，我们将这些文档作为上下文传递给我们的文档链以生成最终的响应。
调用这个链结合了上面概述的两个步骤：
```python
retrieval_chain.invoke(
    {
        "messages": [
            HumanMessage(content="LangSmith 能帮助测试我的 LLM 应用吗？")
        ],
    }
)
```
```output
{'messages': [HumanMessage(content='LangSmith 能帮助测试我的 LLM 应用吗？')],
 'context': [Document(page_content='Skip to main content🦜️🛠️ LangSmith DocsPython DocsJS/TS DocsSearchGo to AppLangSmithOverviewTracingTesting & EvaluationOrganizationsHubLangSmith CookbookOverviewOn this pageLangSmith Overview and User GuideBuilding reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.Over the past two months, we at LangChain', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'})],
 'answer': 'Yes, LangSmith can help test and evaluate your LLM applications. It simplifies the initial setup, and you can use it to monitor your application, log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise.'}
```
是的，LangSmith可以帮助测试和评估您的LLM应用程序。它简化了初始设置，并且您可以使用它来监控应用程序，记录所有跟踪，可视化延迟和令牌使用统计数据，并在出现特定问题时进行故障排除。您还可以点击“在Playground中打开”按钮，访问我们自己构建的Playground，以便在检查LLM调用时尝试更改单词或短语，以查看发生了什么情况。[20]
这是因为检索器没有内在的状态概念，它只会提取与给定查询最相似的文档。为了解决这个问题，我们可以将查询转化为一个独立的查询，不包含任何外部引用的LLM。
下面是一个例子：
```python
from langchain_core.messages import AIMessage, HumanMessage
query_transform_prompt = ChatPromptTemplate.from_messages(
    [
        MessagesPlaceholder(variable_name="messages"),
        (
            "user",
            "给定上述对话，请生成一个搜索查询以查找与对话相关的信息。只回答查询，不要回答其他内容。",
        ),
    ]
)
query_transformation_chain = query_transform_prompt | chat
query_transformation_chain.invoke(
    {
        "messages": [
            HumanMessage(content="LangSmith能帮助测试我的LLM应用吗？"),
            AIMessage(
                content="是的，LangSmith可以帮助测试和评估您的LLM应用。它允许您快速编辑示例并将其添加到数据集中，以扩展评估集的表面积，或者对模型进行微调以提高质量或降低成本。此外，LangSmith还可以用于监视应用程序，记录所有跟踪，可视化延迟和令牌使用统计信息，并在出现特定问题时进行故障排除。"
            ),
            HumanMessage(content="告诉我更多！"),
        ],
    }
)
```
```output
AIMessage(content='"LangSmith LLM应用程序测试和评估"')
```
太棒了！这个转换后的查询将检索与LLM应用程序测试相关的上下文文档。
让我们将其添加到我们的检索链中。我们可以将我们的检索器包装如下：
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnableBranch
query_transforming_retriever_chain = RunnableBranch(
    (
        lambda x: len(x.get("messages", [])) == 1,
        # 如果只有一条消息，我们只需将该消息的内容传递给检索器
        (lambda x: x["messages"][-1].content) | retriever,
    ),
    # 如果有多条消息，我们将输入传递给LLM链以转换查询，然后传递给检索器
    query_transform_prompt | chat | StrOutputParser() | retriever,
).with_config(run_name="chat_retriever_chain")
```
然后，我们可以使用这个查询转换链来使我们的检索链更能够处理这样的后续问题：
```python
SYSTEM_TEMPLATE = """
```
可以，LangSmith可以帮助测试和评估LLM（语言模型）应用程序。它简化了初始设置，并且您可以使用它来监视应用程序、记录所有跟踪、可视化延迟和令牌使用统计数据，并在出现特定问题时进行故障排除。[20]
```markdown
{
  "messages": [
    HumanMessage(content="Can LangSmith help test my LLM applications?"),
    AIMessage(content="Yes, LangSmith can help test and evaluate your LLM applications. It allows you to quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs. Additionally, LangSmith can be used to monitor your application, log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise."),
    HumanMessage(content="Tell me more!")
  ],
  "context": [
    Document(page_content="LangSmith Overview and User Guide | 🦜️🛠️ LangSmith", metadata={"description": "Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.", "language": "en", "source": "https://docs.smith.langchain.com/overview", "title": "LangSmith Overview and User Guide | 🦜️🛠️ LangSmith"}),
    Document(page_content="You can also quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs. Monitoring\u200bAfter all this, your app might finally ready to go in production. LangSmith can also be used to monitor your application in much the same way that you used for debugging. You can log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise. Each run can also be", metadata={"description": "Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.", "language": "en", "source": "https://docs.smith.langchain.com/overview", "title": "LangSmith Overview and User Guide | 🦜️🛠️ LangSmith"}),
    Document(page_content="Skip to main content🦜️🛠️ LangSmith DocsPython DocsJS/TS DocsSearchGo to AppLangSmithOverviewTracingTesting & EvaluationOrganizationsHubLangSmith CookbookOverviewOn this pageLangSmith Overview and User GuideBuilding reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.Over the past two months, we at LangChain", metadata={"description": "Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.", "language": "en", "source": "https://docs.smith.langchain.com/overview", "title": "LangSmith Overview and User Guide | 🦜️🛠️ LangSmith"}),
    Document(page_content="LangSmith makes it easy to manually review and annotate runs through annotation queues.These queues allow you to select any runs based on criteria like model type or automatic evaluation scores, and queue them up for human review. As a reviewer, you can then quickly step through the runs, viewing the input, output, and any existing tags before adding your own feedback.We often use this for a couple of reasons:To assess subjective qualities that automatic evaluators struggle with, like", metadata={"description": "Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.", "language": "en", "source": "https://docs.smith.langchain.com/overview", "title": "LangSmith Overview and User Guide | 🦜️🛠️ LangSmith"})
  ]
}
```
LangSmith简化了构建可靠LLM应用程序的初始设置，但它承认仍然需要努力提高提示、链条和代理的性能，使它们达到足够可靠的水平，可以用于生产。它还提供了手动审查和注释运行的能力，通过注释队列，允许您根据模型类型或自动评估分数等标准选择运行进行人工审查。这个功能特别适用于评估自动评估器难以处理的主观特性。
您可以查看[这个LangSmith跟踪](https://smith.langchain.com/public/bb329a3b-e92a-4063-ad78-43f720fbb5a2/r)来了解内部查询转换步骤。
## 流式处理
因为这个链条是用LCEL构建的，所以您可以使用熟悉的方法，比如`.stream()`：
```python
stream = conversational_retrieval_chain.stream(
    {
        "messages": [
            HumanMessage(content="LangSmith能帮助测试我的LLM应用程序吗？"),
            AIMessage(
                content="是的，LangSmith可以帮助测试和评估您的LLM应用程序。它允许您快速编辑示例并将其添加到数据集中，以扩展评估集的覆盖范围，或者微调模型以提高质量或降低成本。此外，LangSmith还可以用于监视您的应用程序，记录所有跟踪，可视化延迟和标记使用统计信息，并在出现特定问题时进行故障排除。"
            ),
            HumanMessage(content="告诉我更多！"),
        ],
    }
)
for chunk in stream:
    print(chunk)
```
```output
{'messages': [HumanMessage(content='LangSmith能帮助测试我的LLM应用程序吗？'), AIMessage(content='是的，LangSmith可以帮助测试和评估您的LLM应用程序。它允许您快速编辑示例并将其添加到数据集中，以扩展评估集的覆盖范围，或者微调模型以提高质量或降低成本。此外，LangSmith还可以用于监视您的应用程序，记录所有跟踪，可视化延迟和标记使用统计信息，并在出现特定问题时进行故障排除。'), HumanMessage(content='告诉我更多！')]}
```
LangSmith 概述和用户指南 | 🦜️🛠️ LangSmith
构建可靠的LLM应用程序可能具有挑战性。LangChain简化了初始设置，但仍需要进行一些工作，以提高提示、链条和代理的性能水平，使它们足够可靠，可以用于生产环境。[20]
您还可以快速编辑示例并将其添加到数据集中，以扩展评估集的覆盖范围，或微调模型以提高质量或降低成本。监控完成所有这些步骤后，您的应用程序可能终于准备投入生产。LangSmith还可以用于监视您的应用程序，方式与您用于调试的方式非常相似。您可以记录所有跟踪、可视化延迟和令牌使用统计数据，并在问题出现时对特定问题进行故障排除。每次运行也可以。
跳转至主要内容🦜️🛠️ LangSmith 文档Python 文档JS/TS 文档搜索转至应用程序LangSmith概述追踪测试与评估组织中心LangSmith食谱概述在这个页面上LangSmith 概述和用户指南构建可靠的LLM应用程序可能具有挑战性。LangChain简化了初始设置，但仍需要进行一些工作，以提高提示、链条和代理的性能水平，使它们足够可靠，可以用于生产环境。在过去的两个月里，我们在LangChain
LangSmith使手动审查和注释运行变得简单通过注释队列。这些队列允许您根据模型类型或自动评分等标准选择任何运行，并将其排队等待人工审查。作为审阅者，您可以快速浏览运行，查看输入、输出和任何现有标签，然后添加您自己的反馈。我们经常出于几个原因使用这个功能：评估自动评估器难以处理的主观特质，如
这种方法对评估自动评估器难以处理的主观特质尤其有用。
## 进一步阅读
本指南只是对检索技术的浅显介绍。要了解更多有关不同摄取、准备和检索最相关数据的方法，请查阅相关的操作指南[这里](/docs/how_to#document-loaders)。