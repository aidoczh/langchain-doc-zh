[Maritalk](https://www.maritaca.ai)

## 简介

MariTalk 是由巴西公司 [Maritaca AI](https://www.maritaca.ai) 开发的助手。

MariTalk 基于专门训练过的语言模型，能够很好地理解葡萄牙语。

本笔记本演示了如何通过两个示例与 LangChain 一起使用 MariTalk：

1. 使用 MariTalk 执行任务的简单示例。

2. LLM + RAG：第二个示例展示了如何回答问题，答案在一个长文档中，超出了 MariTalk 的令牌限制。为此，我们将使用一个简单的搜索器（BM25）首先搜索最相关的部分，然后将它们提供给 MariTalk 进行回答。

## 安装

首先，使用以下命令安装 LangChain 库（及其所有依赖项）：

```python
!pip install langchain langchain-core langchain-community httpx
```

## API 密钥

您需要一个 API 密钥，可以从 chat.maritaca.ai（"Chaves da API" 部分）获得。

### 示例 1 - 宠物名建议

让我们定义我们的语言模型 ChatMaritalk，并使用您的 API 密钥对其进行配置。

```python
from langchain_community.chat_models import ChatMaritalk
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts.chat import ChatPromptTemplate
llm = ChatMaritalk(
    model="sabia-2-medium",  # 可用模型：sabia-2-small 和 sabia-2-medium
    api_key="",  # 在此处插入您的 API 密钥
    temperature=0.7,
    max_tokens=100,
)
output_parser = StrOutputParser()
chat_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are an assistant specialized in suggesting pet names. Given the animal, you must suggest 4 names.",
        ),
        ("human", "I have a {animal}"),
    ]
)
chain = chat_prompt | llm | output_parser
response = chain.invoke({"animal": "dog"})
print(response)  # 应该回答类似 "1. Max\n2. Bella\n3. Charlie\n4. Rocky"
```

### 流生成

对于涉及生成长文本的任务，例如创建广泛的文章或翻译大型文档，逐部分接收响应，而不是等待完整文本，可能是有利的。这使得应用程序更具响应性和效率，特别是当生成的文本很广泛时。我们提供两种方法来满足这种需求：一种是同步的，另一种是异步的。

#### 同步：

```python
from langchain_core.messages import HumanMessage
messages = [HumanMessage(content="Suggest 3 names for my dog")]
for chunk in llm.stream(messages):
    print(chunk.content, end="", flush=True)
```

#### 异步：

```python
from langchain_core.messages import HumanMessage
async def async_invoke_chain(animal: str):
    messages = [HumanMessage(content=f"Suggest 3 names for my {animal}")]
    async for chunk in llm._astream(messages):
        print(chunk.message.content, end="", flush=True)
await async_invoke_chain("dog")
```

### 示例 2 - RAG + LLM：UNICAMP 2024 入学考试问答系统

对于此示例，我们需要安装一些额外的库：

```python
!pip install unstructured rank_bm25 pdf2image pdfminer-six pikepdf pypdf unstructured_inference fastapi kaleido uvicorn "pillow<10.1.0" pillow_heif -q
```

#### 加载数据库

第一步是使用通知创建包含信息的数据库。为此，我们将从 COMVEST 网站下载通知，并将提取的文本分割成 500 字符窗口。

```python
from langchain_community.document_loaders import OnlinePDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
# 加载 COMVEST 2024 通知
loader = OnlinePDFLoader(
    "https://www.comvest.unicamp.br/wp-content/uploads/2023/10/31-2023-Dispoe-sobre-o-Vestibular-Unicamp-2024_com-retificacao.pdf"
)
data = loader.load()
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500, chunk_overlap=100, separators=["\n", " ", ""]
)
texts = text_splitter.split_documents(data)
```

#### 创建搜索器

现在我们有了数据库，我们需要一个搜索器。在本例中，我们将使用简单的 BM25 作为搜索系统，但这可以被任何其他搜索器（如通过嵌入进行搜索）所替代。

```python
from langchain_community.retrievers import BM25Retriever
retriever = BM25Retriever.from_documents(texts)
```

#### 结合搜索系统 + LLM

现在我们有了搜索器，我们只需要实现一个指定任务的提示，并调用链。

```python
from langchain.chains.question_answering import load_qa_chain
prompt = """Baseado nos seguintes documentos, responda a pergunta abaixo.
{context}
Pergunta: {query}
"""
qa_prompt = ChatPromptTemplate.from_messages([("human", prompt)])
chain = load_qa_chain(llm, chain_type="stuff", verbose=True, prompt=qa_prompt)
query = "Qual o tempo máximo para realização da prova?"
docs = retriever.invoke(query)
chain.invoke(
    {"input_documents": docs, "query": query}
)  # 应该输出类似 "O tempo máximo para realização da prova é de 5 horas."
```

在过去的几十年里，数字图像压缩技术已经取得了巨大的进展。这些技术使得我们能够在不牺牲图像质量的情况下，将图像文件的大小大大减小。其中最常见的压缩格式之一是JPEG（Joint Photographic Experts Group）。

JPEG是一种有损压缩格式，它通过减少图像中的细节和颜色信息来减小文件大小。这种压缩方法基于人眼对颜色和细节的感知能力，利用了人眼对细节和颜色变化的敏感度较低的特点。JPEG压缩算法将图像分成小的8x8像素块，并对每个块进行离散余弦变换（DCT）。然后，通过舍弃高频分量和量化低频分量，来减少图像中的细节和颜色信息。最后，使用熵编码将量化后的数据压缩成更小的文件。

尽管JPEG压缩能够显著减小文件大小，但它也会导致图像质量的损失。这种损失通常表现为图像中的锯齿状边缘、模糊和颜色失真。为了平衡文件大小和图像质量之间的关系，JPEG提供了不同的压缩质量级别。较高的压缩质量级别会导致更小的文件大小，但也会导致更明显的图像质量损失。

除了JPEG，还有其他一些图像压缩格式，如PNG（Portable Network Graphics）和GIF（Graphics Interchange Format）。这些格式使用不同的压缩算法和策略，以适应不同的应用场景和需求。

总的来说，数字图像压缩技术为我们提供了一种有效地减小图像文件大小的方法。然而，我们在使用这些压缩格式时，需要权衡文件大小和图像质量之间的关系，以选择最适合我们需求的压缩质量级别。