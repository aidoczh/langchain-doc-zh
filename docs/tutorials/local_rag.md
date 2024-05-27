# 构建本地 RAG 应用程序
像 [PrivateGPT](https://github.com/imartinez/privateGPT)、[llama.cpp](https://github.com/ggerganov/llama.cpp)、[GPT4All](https://github.com/nomic-ai/gpt4all) 和 [llamafile](https://github.com/Mozilla-Ocho/llamafile) 这样的项目的流行凸显了在本地运行 LLM 的重要性。
LangChain 与许多可以在本地运行的开源 LLM 进行了 [集成](https://integrations.langchain.com/)。
请查看 [这里](/docs/how_to/local_llms) 以获取这些 LLM 的设置说明。
例如，这里我们展示如何使用本地嵌入和本地 LLM 在本地（例如，在您的笔记本电脑上）运行 `GPT4All` 或 `LLaMA2`。
## 文档加载
首先，安装本地嵌入和向量存储所需的软件包。
```python
%pip install --upgrade --quiet  langchain langchain-community langchainhub gpt4all langchain-chroma
```
加载并拆分一个示例文档。
我们将以一篇关于代理的博客文章为例。
```python
from langchain_community.document_loaders import WebBaseLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
loader = WebBaseLoader("https://lilianweng.github.io/posts/2023-06-23-agent/")
data = loader.load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
all_splits = text_splitter.split_documents(data)
```
接下来，以下步骤将在本地下载 `GPT4All` 嵌入（如果您尚未拥有它们）。
```python
from langchain_chroma import Chroma
from langchain_community.embeddings import GPT4AllEmbeddings
vectorstore = Chroma.from_documents(documents=all_splits, embedding=GPT4AllEmbeddings())
```
测试相似性搜索是否可以使用我们的本地嵌入。
```python
question = "任务分解的方法有哪些？"
docs = vectorstore.similarity_search(question)
len(docs)
```
```output
4
```
```python
docs[0]
```
```output
Document(page_content='任务分解可以通过以下方式进行：(1) 使用简单提示的 LLM，例如“XYZ 的步骤。\\n1.”，“实现 XYZ 的子目标是什么？”，(2) 使用任务特定的说明；例如，为写小说而写“写一个故事大纲。”，或者(3) 使用人类输入。', metadata={'description': '以 LLM（大型语言模型）为核心控制器构建代理是一个很酷的概念。一些概念验证演示，如 AutoGPT、GPT-Engineer 和 BabyAGI，是鼓舞人心的例子。LLM 的潜力不仅限于生成写作精美的副本、故事、论文和程序；它可以被构想为一个强大的通用问题解决器。\n代理系统概述在以 LLM 为核心的自主代理系统中，LLM 充当代理的大脑，辅以几个关键组件：', 'language': 'en', 'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/', 'title': "LLM Powered Autonomous Agents | Lil'Log"})
```
## 模型
### LLaMA2
注意：新版本的 `llama-cpp-python` 使用 GGUF 模型文件（参见 [这里](https://github.com/abetlen/llama-cpp-python/pull/633)）。
如果您有现有的 GGML 模型，请查看 [这里](/docs/integrations/llms/llamacpp) 以获取有关转换为 GGUF 的说明。
或者，您可以下载一个已转换为 GGUF 的模型（例如，[这里](https://huggingface.co/TheBloke)）。
最后，如 [这里](/docs/how_to/local_llms) 详细说明的那样安装 `llama-cpp-python`。
```python
%pip install --upgrade --quiet  llama-cpp-python
```
要在 Apple Silicon 上启用 GPU，请按照 [这里](https://github.com/abetlen/llama-cpp-python/blob/main/docs/install/macos.md) 的步骤使用带有 Metal 支持的 Python 绑定。
特别是，请确保 `conda` 正在使用您创建的正确虚拟环境（`miniforge3`）。
例如，对我来说：
```
conda activate /Users/rlm/miniforge3/envs/llama
```
确认后：
```python
! CMAKE_ARGS="-DLLAMA_METAL=on" FORCE_CMAKE=1 /Users/rlm/miniforge3/envs/llama/bin/pip install -U llama-cpp-python --no-cache-dir
```
```python
from langchain_community.llms import LlamaCpp
```
根据 [llama.cpp 文档](/docs/integrations/llms/llamacpp) 中的说明设置模型参数。
```python
n_gpu_layers = 1  # Metal 设置为 1 就足够了。
n_batch = 512  # 应该在 1 和 n_ctx 之间，考虑您的 Apple Silicon 芯片的 RAM 量。
# 确保模型路径对于您的系统是正确的！
llm = LlamaCpp(
    model_path="/Users/rlm/Desktop/Code/llama.cpp/models/llama-2-13b-chat.ggufv3.q4_0.bin",
    n_gpu_layers=n_gpu_layers,
    n_batch=n_batch,
    n_ctx=2048,
    f16_kv=True,  # 必须设置为 True，否则在几次调用后会出现问题
    verbose=True,
)
```
请注意，这些指示表明 [Metal 已正确启用](/docs/integrations/llms/llamacpp)：
```
ggml_metal_init: allocating
ggml_metal_init: using MPS
```
```python
llm.invoke("模拟 Stephen Colbert 和 John Oliver 之间的说唱对战")
```
```output
Llama.generate: prefix-match hit
``````output
by jonathan 
这是一个假想的说唱对战：
[Stephen Colbert]: Yo，我是 Stephen Colbert，以我的喜剧节目而闻名。我在这里要给你的头脑带来一些理智，就像灌肠一样。你的对手？一个笑声和机智话语的人，John Oliver！现在让我们看看谁在互相攻击时能获得最多的笑声
[John Oliver]: Yo，我是 John Oliver，以我自己的喜剧节目而闻名。我要带领你的头脑踏上机智和幽默的冒险。但首先，让我向我们的参赛者介绍一下：Stephen Colbert！他的节目自上世纪九十年代以来一直存在，但现在是时候看看谁能在说唱中胜出
[Stephen Colbert]: 你声称自己是一个机智的人，John Oliver，以你的英国魅力和聪明的言辞。但我的知道我是美国的滑稽人！谁在拿你？没有人！
[John Oliver]: 嘿，Stephen Colbert，不要太自大。你可能
``````output
llama_print_timings:        load time =  4481.74 ms
llama_print_timings:      sample time =   183.05 ms /   256 runs   (    0.72 ms per token,  1398.53 tokens per second)
llama_print_timings: prompt eval time =   456.05 ms /    13 tokens (   35.08 ms per token,    28.51 tokens per second)
llama_print_timings:        eval time =  7375.20 ms /   255 runs   (   28.92 ms per token,    34.58 tokens per second)
llama_print_timings:       total time =  8388.92 ms
```
### GPT4All
同样，我们可以使用 `GPT4All`。
[下载 GPT4All 模型二进制文件](/docs/integrations/llms/gpt4all)。
在 [GPT4All](https://gpt4all.io/index.html) 上的模型资源管理器是选择和下载模型的好方法。
然后，指定你下载的路径。
例如，对我来说，模型存放在这里：
`/Users/rlm/Desktop/Code/gpt4all/models/nous-hermes-13b.ggmlv3.q4_0.bin`
```python
from langchain_community.llms import GPT4All
gpt4all = GPT4All(
    model="/Users/rlm/Desktop/Code/gpt4all/models/nous-hermes-13b.ggmlv3.q4_0.bin",
    max_tokens=2048,
)
```
### llamafile
在本地运行 LLM 的最简单方法之一是使用 [llamafile](https://github.com/Mozilla-Ocho/llamafile)。你只需要：
1) 从 [HuggingFace](https://huggingface.co/models?other=llamafile) 下载一个 llamafile
2) 使文件可执行
3) 运行文件
llamafiles 将模型权重和 [专门编译的](https://github.com/Mozilla-Ocho/llamafile?tab=readme-ov-file#technical-details) `llama.cpp` 版本捆绑到一个单个文件中，可以在大多数计算机上运行，而无需任何额外的依赖。它们还配备了一个嵌入的推理服务器，提供一个 [API](https://github.com/Mozilla-Ocho/llamafile/blob/main/llama.cpp/server/README.md#api-endpoints) 用于与你的模型交互。
下面是一个展示所有 3 个设置步骤的简单 bash 脚本：
```bash
# 从 HuggingFace 下载一个 llamafile
wget https://huggingface.co/jartine/TinyLlama-1.1B-Chat-v1.0-GGUF/resolve/main/TinyLlama-1.1B-Chat-v1.0.Q5_K_M.llamafile
# 使文件可执行。在 Windows 上，只需将文件重命名为以 ".exe" 结尾。
chmod +x TinyLlama-1.1B-Chat-v1.0.Q5_K_M.llamafile
# 启动模型服务器。默认情况下在 http://localhost:8080 监听。
./TinyLlama-1.1B-Chat-v1.0.Q5_K_M.llamafile --server --nobrowser
```
完成上述设置步骤后，你可以通过 LangChain 与模型进行交互：
```python
from langchain_community.llms.llamafile import Llamafile
llamafile = Llamafile()
llamafile.invoke("这是我祖母心爱的意大利面和肉丸的食谱：")
```
```output
'\n-1 1/2 (8 盎司) 牛肉碎，煎炒至不再粉红\n-3 杯全麦意大利面\n-4 (10 盎司) 罐装蒜香番茄和罗勒\n-2 个鸡蛋，打散\n-1 杯帕玛森奶酪碎\n-1/2 茶匙盐\n-1/4 茶匙黑胡椒\n-1 杯面包糠 (16 盎司)\n-2 汤匙橄榄油\n\n烹饪步骤：\n1. 根据包装上的说明煮意大利面。沥干，备用。\n2. 在一个大平底锅中，用中火煎炒牛肉碎，直到不再粉红。倒掉任何多余的油脂。\n3. 加入蒜香番茄和罗勒，用盐和胡椒调味。煮 5 到 7 分钟，或直到酱汁加热。备用。\n4. 在一个大碗中，用叉子或打蛋器打散鸡蛋，直到起泡。加入奶酪、盐和黑胡椒。备用。\n5. 在另一个碗中，混合面包糠和橄榄油。将每根意大利面条浸入鸡蛋混合物，然后裹上面包糠混合物。放在铺有烘焙纸的烤盘上，以防粘连。重复，直到所有意大利面条都裹上面包糠。\n6. 预热烤箱至 375 度。烘烤 18 到 20 分钟，或直到浅金黄色。\n7. 热着时配上肉丸和酱汁。享用！'
```
## 在链中使用
我们可以通过传入检索到的文档和一个简单的提示来创建一个摘要链。
它使用提供的输入键值格式化提示模板，并将格式化的字符串传递给 `GPT4All`、`LLama-V2` 或其他指定的 LLM。
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
# 提示
prompt = PromptTemplate.from_template(
    "总结这些检索到的文档中的主题：{docs}"
)
# 链
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)
chain = {"docs": format_docs} | prompt | llm | StrOutputParser()
# 运行
question = "What are the approaches to Task Decomposition?"
docs = vectorstore.similarity_search(question)
chain.invoke(docs)
```
Llama.generate: 前缀匹配命中
``````output
根据检索到的文档，主要主题包括：
1. 任务分解：将复杂任务分解为较小的子任务的能力，这些子任务可以由 LLM 或代理系统的其他组件处理。
2. LLM 作为核心控制器：将大型语言模型（LLM）用作自主代理系统的主要控制器，辅以其他关键组件，如知识图和规划器。
3. LLM 的潜力：LLM 有潜力被用作强大的通用问题解决器，不仅用于生成写作流畅的文本，还可用于解决复杂任务，实现类似人类智能的效果。
4. 长期规划中的挑战：在规划长期历史和有效探索解决方案空间方面的挑战，这是当前基于 LLM 的自主代理系统的重要局限性。
``````output
llama_print_timings:        加载时间 =  1191.88 毫秒
llama_print_timings:      采样时间 =   134.47 毫秒 /   193 次运行   (    0.70 毫秒每个标记,  1435.25 标记每秒)
llama_print_timings: 提示评估时间 = 39470.18 毫秒 /  1055 标记 (   37.41 毫秒每个标记,    26.73 标记每秒)
llama_print_timings:        评估时间 =  8090.85 毫秒 /   192 次运行   (   42.14 毫秒每个标记,    23.73 标记每秒)
llama_print_timings:       总时间 = 47943.12 毫秒
```
```output
'\n根据检索到的文档，主要主题包括：\n1. 任务分解：将复杂任务分解为较小的子任务的能力，这些子任务可以由 LLM 或代理系统的其他组件处理。\n2. LLM 作为核心控制器：将大型语言模型（LLM）用作自主代理系统的主要控制器，辅以其他关键组件，如知识图和规划器。\n3. LLM 的潜力：LLM 有潜力被用作强大的通用问题解决器，不仅用于生成写作流畅的文本，还可用于解决复杂任务，实现类似人类智能的效果。\n4. 长期规划中的挑战：在规划长期历史和有效探索解决方案空间方面的挑战，这是当前基于 LLM 的自主代理系统的重要局限性。'
```
## 问答
我们还可以使用 LangChain Prompt Hub 存储和获取特定于模型的提示。
让我们尝试使用默认的 RAG 提示，[在这里](https://smith.langchain.com/hub/rlm/rag-prompt)。
```python
from langchain import hub
rag_prompt = hub.pull("rlm/rag-prompt")
rag_prompt.messages
```
```output
[HumanMessagePromptTemplate(prompt=PromptTemplate(input_variables=['context', 'question'], template="You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.\nQuestion: {question} \nContext: {context} \nAnswer:"))]
```
```python
from langchain_core.runnables import RunnablePassthrough, RunnablePick
# Chain
chain = (
    RunnablePassthrough.assign(context=RunnablePick("context") | format_docs)
    | rag_prompt
    | llm
    | StrOutputParser()
)
# Run
chain.invoke({"context": docs, "question": question})
```
```output
Llama.generate: 前缀匹配命中
``````output
任务可以通过将任务分解为较小的子任务来完成，使用简单的提示，如“XYZ 的步骤。”或针对特定任务，如为写小说编写大纲的提示。
``````output
llama_print_timings:        加载时间 = 11326.20 毫秒
llama_print_timings:      采样时间 =    33.03 毫秒 /    47 次运行   (    0.70 毫秒每个标记,  1422.86 标记每秒)
llama_print_timings: 提示评估时间 =  1387.31 毫秒 /   242 标记 (    5.73 毫秒每个标记,   174.44 标记每秒)
llama_print_timings:        评估时间 =  1321.62 毫秒 /    46 次运行   (   28.73 毫秒每个标记,    34.81 标记每秒)
llama_print_timings:       总时间 =  2801.08 毫秒
```
```output
{'output_text': '\n任务可以通过将任务分解为较小的子任务来完成，使用简单的提示，如“XYZ 的步骤。”或针对特定任务，如为写小说编写大纲的提示。'}
```
现在，让我们尝试使用[专门针对 LLaMA 的提示](https://smith.langchain.com/hub/rlm/rag-prompt-llama)，其中[包含特殊标记](https://huggingface.co/blog/llama2#how-to-prompt-llama-2)。
```python
# Prompt
rag_prompt_llama = hub.pull("rlm/rag-prompt-llama")
rag_prompt_llama.messages
```
```output
ChatPromptTemplate(input_variables=['question', 'context'], output_parser=None, partial_variables={}, messages=[HumanMessagePromptTemplate(prompt=PromptTemplate(input_variables=['question', 'context'], output_parser=None, partial_variables={}, template="[INST]<<SYS>> 你是一个用于问答任务的助手。使用以下检索到的上下文片段来回答问题。如果你不知道答案，只需说你不知道。最多使用三句话，保持回答简洁。<</SYS>> \n问题: {question} \n上下文: {context} \n回答: [/INST]", template_format='f-string', validate_template=True), additional_kwargs={})])
```
```python
# 链
chain = (
    RunnablePassthrough.assign(context=RunnablePick("context") | format_docs)
    | rag_prompt_llama
    | llm
    | StrOutputParser()
)
# 运行
chain.invoke({"context": docs, "question": question})
```
```output
Llama.generate: 前缀匹配成功
``````output
  当然，我很乐意帮忙！基于上下文，以下是一些任务：
1. 使用简单提示的大语言模型 (LLM)：使用简单提示，如“XYZ 的步骤”或“实现 XYZ 的子目标是什么？”，将任务分解为更小的步骤。
2. 任务特定：另一个方法是使用任务特定的提示，比如为写小说写大纲，来引导任务的完成。
3. 人类输入：在任务需要高度创造力或专业知识的情况下，可以使用人类输入来补充过程。
在长期任务和任务规划方面，一个主要问题是，当面临错误时，LLMs 需要调整计划，使其不太适应通过反复试验学习的人类。
``````output
llama_print_timings:        加载时间 = 11326.20 毫秒
llama_print_timings:      采样时间 =   144.81 毫秒 /   207 次运行   (    0.70 毫秒每标记,  1429.47 标记每秒)
llama_print_timings: 提示评估时间 =  1506.13 毫秒 /   258 标记 (    5.84 毫秒每标记,   171.30 标记每秒)
llama_print_timings:        评估时间 =  6231.92 毫秒 /   206 次运行   (   30.25 毫秒每标记,    33.06 标记每秒)
llama_print_timings:       总时间 =  8158.41 毫秒
```
```output
{'output_text': '  当然，我很乐意帮忙！基于上下文，以下是一些任务：\n\n1. 使用简单提示的大语言模型 (LLM)：使用简单提示，如“XYZ 的步骤”或“实现 XYZ 的子目标是什么？”，将任务分解为更小的步骤。\n2. 任务特定：另一个方法是使用任务特定的提示，比如为写小说写大纲，来引导任务的完成。\n3. 人类输入：在任务需要高度创造力或专业知识的情况下，可以使用人类输入来补充过程。\n\n在长期任务和任务规划方面，一个主要问题是，当面临错误时，LLMs 需要调整计划，使其不太适应通过反复试验学习的人类。'}
```
## 带检索的问答
我们可以自动从向量存储中检索用户问题相关的文档，而不是手动传入文档。
这将使用默认的 QA 提示 (在 [这里](https://github.com/langchain-ai/langchain/blob/275b926cf745b5668d3ea30236635e20e7866442/langchain/chains/retrieval_qa/prompt.py#L4) 中显示)，并将从向量数据库中检索。
```python
retriever = vectorstore.as_retriever()
qa_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | rag_prompt
    | llm
    | StrOutputParser()
)
```
```python
qa_chain.invoke(question)
```
```output
Llama.generate: 前缀匹配成功
``````output
  当然！基于上下文，这是我的回答:
有几种任务分解方法，包括:
1. 基于LLM的简单提示，例如“XYZ 的步骤”或“实现 XYZ 的子目标是什么?”
2. 任务特定，比如为写小说写大纲。
3. 人类输入以指导过程。
这些方法可用于将复杂任务分解为更小、更易管理的子任务，有助于提高任务的效率和有效性。然而，在长期任务和规划中，由于需要在漫长历史上进行计划并探索空间，LLMs可能需要在面临错误时调整计划，使其对通过试错学习的人类学习者不够稳健。
``````output
llama_print_timings:        加载时间 = 11326.20 毫秒
llama_print_timings:      采样时间 =   139.20 毫秒 /   200 次运行   (    0.70 毫秒每标记,  1436.76 标记每秒)
llama_print_timings: 提示评估时间 =  1532.26 毫秒 /   258 标记 (    5.94 毫秒每标记,   168.38 标记每秒)
llama_print_timings:        评估时间 =  5977.62 毫秒 /   199 次运行   (   30.04 毫秒每标记,    33.29 标记每秒)
llama_print_timings:       总时间 =  7916.21 毫秒
```
```output
{'query': '任务分解的方法是什么？',
 'result': '  当然！基于上下文，这是我的回答:\n\n有几种任务分解方法，包括:\n\n1. 基于LLM的简单提示，例如“XYZ 的步骤”或“实现 XYZ 的子目标是什么?”\n2. 任务特定，比如为写小说写大纲。\n3. 人类输入以指导过程。\n\n这些方法可用于将复杂任务分解为更小、更易管理的子任务，有助于提高任务的效率和有效性。然而，在长期任务和规划中，由于需要在漫长历史上进行计划并探索空间，LLMs可能需要在面临错误时调整计划，使其对通过试错学习的人类学习者不够稳健。'}
```