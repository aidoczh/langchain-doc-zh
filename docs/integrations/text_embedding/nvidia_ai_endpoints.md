# NVIDIA AI Foundation Endpoints 

[NVIDIA AI Foundation Endpoints](https://www.nvidia.com/en-us/ai-data-science/foundation-models/) 提供了便捷的方式让用户访问 NVIDIA 托管的 API 端点，用于访问 NVIDIA AI 基础模型，如 Mixtral 8x7B、Llama 2、Stable Diffusion 等。这些模型托管在[NVIDIA API 目录](https://build.nvidia.com/)上，经过优化、测试，并在 NVIDIA AI 平台上托管，使其快速且易于评估，进一步定制，并在任何加速堆栈上以最佳性能无缝运行。

通过[NVIDIA AI Foundation Endpoints](https://www.nvidia.com/en-us/ai-data-science/foundation-models/)，您可以从在[NVIDIA DGX Cloud](https://www.nvidia.com/en-us/data-center/dgx-cloud/)上运行的完全加速堆栈中快速获得结果。一旦定制，这些模型可以在任何地方部署，并使用[ NVIDIA AI Enterprise](https://www.nvidia.com/en-us/data-center/products/ai-enterprise/)提供企业级安全性、稳定性和支持。

这些模型可以通过[`langchain-nvidia-ai-endpoints`](https://pypi.org/project/langchain-nvidia-ai-endpoints/)包轻松访问，如下所示。

这个示例介绍了如何使用 LangChain 与支持的[NVIDIA Retrieval QA Embedding Model](https://build.nvidia.com/nvidia/embed-qa-4)进行交互，用于[retrieval-augmented generation](https://developer.nvidia.com/blog/build-enterprise-retrieval-augmented-generation-apps-with-nvidia-retrieval-qa-embedding-model/)，通过`NVIDIAEmbeddings`类。

有关通过此 API 访问聊天模型的更多信息，请查看[ChatNVIDIA](https://python.langchain.com/docs/integrations/chat/nvidia_ai_endpoints/)文档。

## 安装

```python
%pip install --upgrade --quiet  langchain-nvidia-ai-endpoints
```

## 设置

**开始使用：**

1. 在[NVIDIA](https://build.nvidia.com/)上创建免费帐户，该网站托管 NVIDIA AI Foundation 模型。

2. 选择`Retrieval`选项卡，然后选择您选择的模型。

3. 在`输入`下选择`Python`选项卡，然后点击`获取 API 密钥`。然后点击`生成密钥`。

4. 复制并保存生成的密钥为`NVIDIA_API_KEY`。从那里，您应该可以访问端点。

```python
import getpass
import os
# del os.environ['NVIDIA_API_KEY']  ## 删除密钥并重置
if os.environ.get("NVIDIA_API_KEY", "").startswith("nvapi-"):
    print("有效的 NVIDIA_API_KEY 已存在于环境中。删除以重置")
else:
    nvapi_key = getpass.getpass("NVAPI 密钥（以 nvapi- 开头）：")
    assert nvapi_key.startswith("nvapi-"), f"{nvapi_key[:5]}... 不是有效的密钥"
    os.environ["NVIDIA_API_KEY"] = nvapi_key
```

我们应该能在列表中看到一个嵌入模型，它可以与 LLM 结合使用，以实现有效的 RAG 解决方案。我们可以很容易地使用`NVIDIAEmbeddings`模型与这个模型进行交互。

## 初始化

在初始化嵌入模型时，您可以通过传递模型来选择模型，例如下面的`ai-embed-qa-4`，或者通过不传递任何参数来使用默认值。

```python
from langchain_nvidia_ai_endpoints import NVIDIAEmbeddings
embedder = NVIDIAEmbeddings(model="ai-embed-qa-4")
```

这个模型是一个经过微调的 E5-large 模型，支持预期的`Embeddings`方法，包括：

- `embed_query`：为查询样本生成查询嵌入。

- `embed_documents`：为您想要搜索的文档列表生成段落嵌入。

- `aembed_quey`/`embed_documents`：上述方法的异步版本。

### **相似度**

以下是对这些数据点相似度的快速测试：

**查询：**

- 在科姆恰特卡的天气怎么样？

- 意大利以什么食物而闻名？

- 我的名字是什么？我打赌你不记得...

- 生活的意义到底是什么？

- 生活的意义在于快乐:D

**文档：**

- 科姆恰特卡的天气寒冷，冬季漫长而严寒。

- 意大利以意大利面、比萨饼、意式冰淇淋和浓缩咖啡而闻名。

- 我无法记住个人姓名，只能提供信息。

- 生活的目的各不相同，通常被视为个人的满足。

- 享受生活的时刻确实是一种美妙的方式。

### 嵌入运行时间

```python
print("\n顺序嵌入：")
q_embeddings = [
    embedder.embed_query("在科姆恰特卡的天气怎么样？"),
    embedder.embed_query("意大利以什么食物而闻名？"),
    embedder.embed_query("我的名字是什么？我打赌你不记得..."),
    embedder.embed_query("生活的意义到底是什么？"),
    embedder.embed_query("生活的意义在于快乐:D"),
]
print("形状:", (len(q_embeddings), len(q_embeddings[0])))
```

### 文档嵌入

```python
print("\n批量文档嵌入：")
d_embeddings = embedder.embed_documents(
    [
        "科姆恰特卡的天气寒冷，冬季漫长而严寒。",
        "意大利以意大利面、比萨饼、意式冰淇淋和浓缩咖啡而闻名。",
        "我无法记住个人姓名，只能提供信息。",
        "生活的目的各不相同，通常被视为个人的满足。",
        "享受生活的时刻确实是一种美妙的方式。",
    ]
)
print("形状:", (len(q_embeddings), len(q_embeddings[0])))
```

现在我们已经生成了嵌入向量，我们可以对结果进行简单的相似性检查，看看哪些文档在检索任务中会被视为合理的答案：

```python
%pip install --upgrade --quiet  matplotlib scikit-learn
```

```python
import matplotlib.pyplot as plt
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity
# 计算查询嵌入向量和文档嵌入向量之间的相似性矩阵
cross_similarity_matrix = cosine_similarity(
    np.array(q_embeddings),
    np.array(d_embeddings),
)
# 绘制相似性矩阵
plt.figure(figsize=(8, 6))
plt.imshow(cross_similarity_matrix, cmap="Greens", interpolation="nearest")
plt.colorbar()
plt.title("相似性矩阵")
plt.xlabel("查询嵌入向量")
plt.ylabel("文档嵌入向量")
plt.grid(True)
plt.show()
```

作为提醒，发送到我们系统的查询和文档如下：

**查询：**

- 堪察加的天气如何？

- 意大利以什么食物而闻名？

- 我的名字是什么？我打赌你不记得了…

- 生活的意义到底是什么？

- 生活的意义在于享乐 :D

**文档：**

- 堪察加的天气寒冷，冬季漫长严寒。

- 意大利以意大利面、比萨、意式冰淇淋和浓缩咖啡而闻名。

- 我无法回忆个人姓名，只能提供信息。

- 生活的目的各不相同，通常被视为个人的实现。

- 享受生活的时刻确实是一种美妙的方式。

## 截断

嵌入模型通常具有固定的上下文窗口，用于确定可以嵌入的最大输入标记数。此限制可以是硬限制，等于模型的最大输入标记长度，也可以是有效限制，超出该限制后嵌入的准确性会降低。

由于模型操作标记，而应用程序通常使用文本，因此对于应用程序来确保其输入保持在模型的标记限制内可能是具有挑战性的。默认情况下，如果输入过大，将引发异常。

为了帮助解决这个问题，NVIDIA 的 NIMs（API 目录或本地）提供了一个 `truncate` 参数，如果输入过大，服务器端会对其进行截断。

`truncate` 参数有三个选项：

 - "NONE"：默认选项。如果输入过大，将引发异常。

 - "START"：服务器从开头（左侧）对输入进行截断，必要时丢弃标记。

 - "END"：服务器从末尾（右侧）对输入进行截断，必要时丢弃标记。

```python
long_text = "AI is amazing, amazing is " * 100
```

```python
strict_embedder = NVIDIAEmbeddings()
try:
    strict_embedder.embed_query(long_text)
except Exception as e:
    print("错误:", e)
```

```python
truncating_embedder = NVIDIAEmbeddings(truncate="END")
truncating_embedder.embed_query(long_text)[:5]
```

## RAG 检索

以下是对 [LangChain 表达语言检索手册条目](https://python.langchain.com/docs/expression_language/cookbook/retrieval) 的初始示例的重新利用，但使用了 AI Foundation Models 的 [Mixtral 8x7B Instruct](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/ai-foundation/models/mixtral-8x7b) 和 [NVIDIA Retrieval QA Embedding](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/ai-foundation/models/nvolve-40k) 模型，在它们的游乐场环境中可用。手册中的后续示例也如预期般运行良好，我们鼓励您尝试这些选项。

**提示：** 我们建议在内部推理（例如遵循指令进行数据提取、工具选择等）时使用 Mixtral，而在最终的“根据历史和上下文为用户制作简单响应”的单个最终响应中使用 Llama-Chat。

```python
%pip install --upgrade --quiet  langchain faiss-cpu tiktoken
from operator import itemgetter
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_nvidia_ai_endpoints import ChatNVIDIA
```

```python
vectorstore = FAISS.from_texts(
    ["harrison worked at kensho"],
    embedding=NVIDIAEmbeddings(model="ai-embed-qa-4"),
)
retriever = vectorstore.as_retriever()
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "仅基于以下上下文回答：\n<Documents>\n{context}\n</Documents>",
        ),
        ("user", "{question}"),
    ]
)
model = ChatNVIDIA(model="ai-mixtral-8x7b-instruct")
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)
chain.invoke("harrison 在哪工作？")
```

```python

prompt = ChatPromptTemplate.from_messages(

    [

        (

            "system",

"根据以下上下文信息回答问题：\n<文档>\n{context}\n</文档>"

"\n只能使用以下语言回答：{language}",

),

("用户", "{question}"),

]

)

链式调用 = (

{

"context": itemgetter("question") | retriever,

"question": itemgetter("question"),

"language": itemgetter("language"),

}

| prompt

| model

| StrOutputParser()

)

链式调用.invoke({"question": "哈里森在哪里工作", "language": "意大利语"})