

<a target="_blank" href="https://colab.research.google.com/github/langchain-ai/langchain/blob/master/docs/docs/integrations/callbacks/uptrain.ipynb">

  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>

</a>

# UpTrain

> UpTrain [[github](https://github.com/uptrain-ai/uptrain) || [website](https://uptrain.ai/) || [docs](https://docs.uptrain.ai/getting-started/introduction)] 是一个开源平台，用于评估和改进LLM应用程序。它提供了20多个预配置检查的评分（涵盖语言、代码、嵌入使用案例），对失败案例进行根本原因分析，并提供解决方案的指导。

## UpTrain 回调处理程序

这个笔记本展示了UpTrain回调处理程序如何无缝集成到您的流程中，促进多样化的评估。我们选择了一些我们认为适合评估链条的评估。这些评估会自动运行，并在输出中显示结果。有关UpTrain评估的更多详细信息，请参见[这里](https://github.com/uptrain-ai/uptrain?tab=readme-ov-file#pre-built-evaluations-we-offer-)。

Langchain 中选择的检索器用于演示如下：

### 1. **Vanilla RAG**:

RAG 在检索上下文和生成响应中起着至关重要的作用。为了确保其性能和响应质量，我们进行以下评估：

- **[上下文相关性](https://docs.uptrain.ai/predefined-evaluations/context-awareness/context-relevance)**: 确定从查询中提取的上下文是否与响应相关。

- **[事实准确性](https://docs.uptrain.ai/predefined-evaluations/context-awareness/factual-accuracy)**: 评估LLM是否在产生幻觉或提供不正确的信息。

- **[响应完整性](https://docs.uptrain.ai/predefined-evaluations/response-quality/response-completeness)**: 检查响应是否包含查询请求的所有信息。

### 2. **多查询生成**:

MultiQueryRetriever 创建原问题含义相似的多个变体问题。考虑到复杂性，我们包括了先前的评估，并添加了：

- **[多查询准确性](https://docs.uptrain.ai/predefined-evaluations/query-quality/multi-query-accuracy)**: 确保生成的多个查询与原始查询意思相同。

### 3. **上下文压缩和重新排序**:

重新排序涉及根据与查询相关性重新排序节点并选择前n个节点。重新排序完成后，节点数量可能会减少，因此我们进行以下评估：

- **[上下文重新排序](https://docs.uptrain.ai/predefined-evaluations/context-awareness/context-reranking)**: 检查重新排序后的节点顺序是否比原始顺序更相关于查询。

- **[上下文简洁性](https://docs.uptrain.ai/predefined-evaluations/context-awareness/context-conciseness)**: 检查减少的节点数量是否仍然提供所有所需的信息。

这些评估共同确保链条中的RAG、MultiQueryRetriever和重新排序过程的稳健性和有效性。

## 安装依赖库

```python
%pip install -qU langchain langchain_openai uptrain faiss-cpu flashrank
```
```output
huggingface/tokenizers: The current process just got forked, after parallelism has already been used. Disabling parallelism to avoid deadlocks...
To disable this warning, you can either:
	- Avoid using `tokenizers` before the fork if possible
	- Explicitly set the environment variable TOKENIZERS_PARALLELISM=(true | false)
``````output
WARNING: There was an error checking the latest version of pip.
Note: you may need to restart the kernel to use updated packages.
```

注意：如果要使用启用了GPU的库版本，您也可以安装`faiss-gpu`，而不是`faiss-cpu`。

## 导入库

```python
from getpass import getpass
from langchain.chains import RetrievalQA
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import FlashrankRerank
from langchain.retrievers.multi_query import MultiQueryRetriever
from langchain_community.callbacks.uptrain_callback import UpTrainCallbackHandler
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers.string import StrOutputParser
from langchain_core.prompts.chat import ChatPromptTemplate
from langchain_core.runnables.passthrough import RunnablePassthrough
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import (
    RecursiveCharacterTextSplitter,
)
```

## 加载文档

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
```

## 将文档分割成块

```python
text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
chunks = text_splitter.split_documents(documents)
## 创建检索器
```python

embeddings = OpenAIEmbeddings()

db = FAISS.from_documents(chunks, embeddings)

retriever = db.as_retriever()

```
## 定义 LLM
```python
llm = ChatOpenAI(temperature=0, model="gpt-4")
```
## 设置
UpTrain 为您提供：
1. 具有高级钻取和过滤选项的仪表板
2. 在失败案例中的见解和共同主题
3. 对生产数据的可观察性和实时监控
4. 通过与您的 CI/CD 流水线无缝集成进行回归测试
您可以选择以下选项之一来使用 UpTrain 进行评估：
### 1. **UpTrain 的开源软件 (OSS)**：
您可以使用开源评估服务来评估您的模型。在这种情况下，您需要提供 OpenAI API 密钥。UpTrain 使用 GPT 模型来评估 LLM 生成的响应。您可以在[这里](https://platform.openai.com/account/api-keys)获取您的 API 密钥。
为了在 UpTrain 仪表板中查看您的评估结果，您需要通过在终端中运行以下命令来设置它：
```bash

git clone https://github.com/uptrain-ai/uptrain

cd uptrain

bash run_uptrain.sh

```
这将在您的本地机器上启动 UpTrain 仪表板。您可以在 `http://localhost:3000/dashboard` 上访问它。
参数：
- key_type="openai"
- api_key="OPENAI_API_KEY"
- project_name="PROJECT_NAME"
### 2. **UpTrain 托管服务和仪表板**：
或者，您可以使用 UpTrain 的托管服务来评估您的模型。您可以在[这里](https://uptrain.ai/)创建免费的 UpTrain 账户并获得免费试用积分。如果您需要更多的试用积分，可以在[这里](https://calendly.com/uptrain-sourabh/30min)与 UpTrain 的维护者预约电话。
使用托管服务的好处包括：
1. 无需在本地机器上设置 UpTrain 仪表板。
2. 可以访问许多 LLM 而无需它们的 API 密钥。
一旦您执行评估，您可以在 UpTrain 仪表板上查看它们，网址为 `https://dashboard.uptrain.ai/dashboard`。
参数：
- key_type="uptrain"
- api_key="UPTRAIN_API_KEY"
- project_name="PROJECT_NAME"
**注意：** `project_name` 将是在 UpTrain 仪表板中显示执行评估的项目名称。
## 设置 API 密钥
笔记本将提示您输入 API 密钥。您可以通过在下面的单元格中更改 `key_type` 参数来选择 OpenAI API 密钥或 UpTrain API 密钥。
```python
KEY_TYPE = "openai"  # or "uptrain"
API_KEY = getpass()
```
# 1. 原始 RAG
UpTrain 回调处理程序将自动捕获生成的查询、上下文和响应，并对响应运行以下三个评估 *(从 0 到 1 分)*：
- **[上下文相关性](https://docs.uptrain.ai/predefined-evaluations/context-awareness/context-relevance)**：检查从查询中提取的上下文是否与响应相关。
- **[事实准确性](https://docs.uptrain.ai/predefined-evaluations/context-awareness/factual-accuracy)**：检查响应的事实准确性。
- **[响应完整性](https://docs.uptrain.ai/predefined-evaluations/response-quality/response-completeness)**：检查响应是否包含查询所需的所有信息。
```python
# 创建 RAG 提示
template = """基于以下上下文回答问题，上下文可以包括文本和表格：
{context}
问题：{question}
"""
rag_prompt_text = ChatPromptTemplate.from_template(template)
# 创建链
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | rag_prompt_text
    | llm
    | StrOutputParser()
)
# 创建 UpTrain 回调处理程序
uptrain_callback = UpTrainCallbackHandler(key_type=KEY_TYPE, api_key=API_KEY)
config = {"callbacks": [uptrain_callback}
# 使用查询调用链
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = chain.invoke(query, config=config)
```
```output

2024-04-17 17:03:44.969 | INFO     | uptrain.framework.evalllm:evaluate_on_server:378 - 发送行 0 到 <50 的评估请求到 Uptrain

2024-04-17 17:04:05.809 | INFO     | uptrain.framework.evalllm:evaluate:367 - 本地服务器未运行，请启动服务器以记录数据并在仪表板中可视化！

```
回应：总统提到他在4天前提名了Ketanji Brown Jackson担任美国最高法院的法官。他形容她是全国顶尖的法律智囊，将继续布雷耶法官卓越的传统。他还提到她曾是一位顶尖的私人执业诉讼律师，曾担任联邦公共辩护律师，并来自一家公立学校教育工作者和警察的家庭。他形容她是一位建立共识的人，并指出自提名以来，她得到了各方支持，包括警察兄弟会和被民主党和共和党任命的前法官。
上下文相关性得分：1.0
事实准确性得分：1.0
回应完整性得分：1.0
# 2. 多查询生成
**MultiQueryRetriever** 用于解决 RAG pipeline 可能无法根据查询返回最佳文档集的问题。它生成与原始查询意思相同的多个查询，然后为每个查询获取文档。
为了评估这个检索器，UpTrain 将进行以下评估：
- **[多查询准确性](https://docs.uptrain.ai/predefined-evaluations/query-quality/multi-query-accuracy)**：检查生成的多个查询是否与原始查询意思相同。
```python
# 创建检索器
multi_query_retriever = MultiQueryRetriever.from_llm(retriever=retriever, llm=llm)
# 创建 UpTrain 回调
uptrain_callback = UpTrainCallbackHandler(key_type=KEY_TYPE, api_key=API_KEY)
config = {"callbacks": [uptrain_callback]}
# 创建 RAG 提示
模板 = """根据以下上下文回答问题，上下文可以包括文本和表格：
{context}
问题：{question}
"""
rag_prompt_text = ChatPromptTemplate.from_template(template)
chain = (
    {"context": multi_query_retriever, "question": RunnablePassthrough()}
    | rag_prompt_text
    | llm
    | StrOutputParser()
)
# 使用查询调用链
问题 = "总统对Ketanji Brown Jackson有何评论"
文档 = chain.invoke(问题, config=config)
```
```output

2024-04-17 17:04:10.675 | INFO     | uptrain.framework.evalllm:evaluate_on_server:378 - 发送行0到<50的评估请求至 Uptrain

2024-04-17 17:04:16.804 | INFO     | uptrain.framework.evalllm:evaluate:367 - 本地服务器未运行，请启动服务器以记录数据并在仪表板中可视化！

``````output

问题：总统对Ketanji Brown Jackson有何评论

多个查询：

  - 总统如何评论Ketanji Brown Jackson？

  - 总统对Ketanji Brown Jackson有哪些言论？

  - 总统对Ketanji Brown Jackson做出了什么声明？

多查询准确性得分：0.5

``````output

2024-04-17 17:04:22.027 | INFO     | uptrain.framework.evalllm:evaluate_on_server:378 - 发送行0到<50的评估请求至 Uptrain

2024-04-17 17:04:44.033 | INFO     | uptrain.framework.evalllm:evaluate:367 - 本地服务器未运行，请启动服务器以记录数据并在仪表板中可视化！

``````output

问题：总统对Ketanji Brown Jackson有何评论

回应：总统提到他在4天前提名了联邦上诉法院法官Ketanji Brown Jackson担任美国最高法院的法官。他形容她是全国顶尖的法律智囊，将继续布雷耶法官卓越的传统。他还提到自提名以来，她得到了广泛支持，从警察兄弟会到被民主党和共和党任命的前法官。

上下文相关性得分：1.0

事实准确性得分：1.0

回应完整性得分：1.0

```
# 3. 上下文压缩和重新排序
重新排序过程涉及根据与查询相关性重新排列节点并选择前 n 个节点。由于重新排序完成后节点数量可能减少，我们进行以下评估：
- **[上下文重新排序](https://docs.uptrain.ai/predefined-evaluations/context-awareness/context-reranking)**：检查重新排序后的节点顺序是否比原始顺序更相关于查询。
- **[上下文简洁性](https://docs.uptrain.ai/predefined-evaluations/context-awareness/context-conciseness)**：检查减少的节点数量是否仍提供所有所需信息。
```python
# 创建检索器
compressor = FlashrankRerank()
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor, base_retriever=retriever
)
# 创建调用链
chain = RetrievalQA.from_chain_type(llm=llm, retriever=compression_retriever)
# 创建 UpTrain 回调
uptrain_callback = UpTrainCallbackHandler(key_type=KEY_TYPE, api_key=API_KEY)
config = {"callbacks": [uptrain_callback]}
# 使用查询调用链
查询 = "总统对Ketanji Brown Jackson有何评论"
```python
result = chain.invoke(query, config=config)
```
```output
2024-04-17 17:04:46.462 | INFO     | uptrain.framework.evalllm:evaluate_on_server:378 - Sending evaluation request for rows 0 to <50 to the Uptrain
2024-04-17 17:04:53.561 | INFO     | uptrain.framework.evalllm:evaluate:367 - Local server not running, start the server to log data and visualize in the dashboard!
``````output
问题：总统对凯坦吉·布朗·杰克逊(Ketanji Brown Jackson)有何说法
上下文简洁度分数：0.0
上下文重新排序分数：1.0
``````output
2024-04-17 17:04:56.947 | INFO     | uptrain.framework.evalllm:evaluate_on_server:378 - Sending evaluation request for rows 0 to <50 to the Uptrain
2024-04-17 17:05:16.551 | INFO     | uptrain.framework.evalllm:evaluate:367 - Local server not running, start the server to log data and visualize in the dashboard!
``````output
问题：总统对凯坦吉·布朗·杰克逊有何说法
回答：总统提到他提名了巡回上诉法院法官凯坦吉·布朗·杰克逊(Ketanji Brown Jackson)担任美国最高法院大法官，这是4天前的事。他形容她是全国顶尖的法律智囊，将继续布雷耶法官卓越的传统。
上下文相关性分数：1.0
事实准确性分数：1.0
回答完整性分数：0.5
```

# UpTrain 的仪表板和见解

以下是一个展示仪表板和见解的简短视频：

![langchain_uptrain.gif](https://uptrain-assets.s3.ap-south-1.amazonaws.com/images/langchain/langchain_uptrain.gif)

```