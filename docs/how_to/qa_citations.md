# 如何让 RAG 应用程序添加引用

本指南介绍了获取模型引用其生成响应所引用的源文件部分的方法。

我们将涵盖五种方法：

1. 使用工具调用引用文档 ID；

2. 使用工具调用引用文档 ID 并提供文本片段；

3. 直接提示；

4. 检索后处理（即，压缩检索到的内容，使其更相关）；

5. 生成后处理（即，发出第二个 LLM 调用，以注释生成的答案引用）。

我们通常建议使用适合您用例的列表中的第一项。也就是说，如果您的模型支持工具调用，请尝试方法 1 或 2；否则，或者如果这些方法失败，请继续往下执行列表。

首先让我们创建一个简单的 RAG 链。首先，我们将从维基百科中检索，使用 [WikipediaRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.wikipedia.WikipediaRetriever.html)。

## 设置

首先，我们需要安装一些依赖项并设置我们将使用的模型的环境变量。

```python
%pip install -qU langchain langchain-openai langchain-anthropic langchain-community wikipedia
```

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
os.environ["ANTHROPIC_API_KEY"] = getpass.getpass()
# 如果要记录到 LangSmith，请取消注释以下代码
# os.environ["LANGCHAIN_TRACING_V2"] = "true
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

让我们首先选择一个 LLM：

```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs customVarName="llm" />
```

```python
from langchain_community.retrievers import WikipediaRetriever
from langchain_core.prompts import ChatPromptTemplate
system_prompt = (
    "You're a helpful AI assistant. Given a user question "
    "and some Wikipedia article snippets, answer the user "
    "question. If none of the articles answer the question, "
    "just say you don't know."
    "\n\nHere are the Wikipedia articles: "
    "{context}"
)
retriever = WikipediaRetriever(top_k_results=6, doc_content_chars_max=2000)
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", "{input}"),
    ]
)
prompt.pretty_print()
```

```output
================================ 系统消息 =================================
您是一个乐于助人的 AI 助手。给定一个用户问题和一些维基百科文章片段，回答用户问题。如果没有一篇文章回答了问题，就说您不知道。
这里是维基百科文章：{context}
================================ 用户消息 =================================
{input}
```

现在我们有了一个模型、检索器和提示，让我们将它们全部链接在一起。我们需要添加一些逻辑，将我们检索到的文档格式化为一个字符串，以便传递给我们的提示。根据 [向 RAG 应用程序添加引用](/docs/how_to/qa_citations) 的指南，我们将使我们的链返回答案和检索到的文档。

```python
from typing import List
from langchain_core.documents import Document
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
def format_docs(docs: List[Document]):
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
```

```python
result = chain.invoke({"input": "How fast are cheetahs?"})
```

```python
print(result.keys())
```

```output
dict_keys(['input', 'context', 'answer'])
```

```python
print(result["context"][0])
```

```output
page_content='The cheetah (Acinonyx jubatus) is a large cat and the fastest land animal. It has a tawny to creamy white or pale buff fur that is marked with evenly spaced, solid black spots. The head is small and rounded, with a short snout and black tear-like facial streaks. It reaches 67–94 cm (26–37 in) at the shoulder, and the head-and-body length is between 1.1 and 1.5 m (3 ft 7 in and 4 ft 11 in). Adults weigh between 21 and 72 kg (46 and 159 lb). The cheetah is capable of running at 93 to 104 km/h (58 to 65 mph); it has evolved specialized adaptations for speed, including a light build, long thin legs and a long tail.\nThe cheetah was first described in the late 18th century. Four subspecies are recognised today that are native to Africa and central Iran. An African subspecies was introduced to India in 2022. It is now distributed mainly in small, fragmented populations in northwestern, eastern and southern Africa and central Iran. It lives in a variety of habitats such as savannahs in the Serengeti, arid mountain ranges in the Sahara, and hilly desert terrain.\nThe cheetah lives in three main social groups: females and their cubs, male "coalitions", and solitary males. While females lead a nomadic life searching for prey in large home ranges, males are more sedentary and instead establish much smaller territories in areas with plentiful prey and access to females. The cheetah is active during the day, with peaks during dawn and dusk. It feeds on small- to medium-sized prey, mostly weighing under 40 kg (88 lb), and prefers medium-sized ungulates such as impala, springbok and Thomson\'s gazelles. The cheetah typically stalks its prey within 60–100 m (200–330 ft) before charging towards it, trips it during the chase and bites its throat to suffocate it to death. It breeds throughout the year. After a gestation of nearly three months, females give birth to a litter of three or four cubs. Cheetah cubs are highly vulnerable to predation by other large carnivores. They are weaned a' metadata={'title': 'Cheetah', 'summary': 'The cheetah (Acinonyx jubatus) is a large cat and the fastest land animal. It has a tawny to creamy white or pale buff fur that is marked with evenly spaced, solid black spots. The head is small and rounded, with a short snout and black tear-like facial streaks. It reaches 67–94 cm (26–37 in) at the shoulder, and the head-and-body length is between 1.1 and 1.5 m (3 ft 7 in and 4 ft 11 in). Adults weigh between 21 and 72 kg (46 and 159 lb). The cheetah is capable of running at 93 to 104 km/h (58 to 65 mph); it has evolved specialized adaptations for speed, including a light build, long thin legs and a long tail.\nThe cheetah was first described in the late 18th century. Four subspecies are recognised today that are native to Africa and central Iran. An African subspecies was introduced to India in 2022. It is now distributed mainly in small, fragmented populations in northwestern, eastern and southern Africa and central Iran. It lives in a variety of habitats such as savannahs in the Serengeti, arid mountain ranges in the Sahara, and hilly desert terrain.\nThe cheetah lives in three main social groups: females and their cubs, male "coalitions", and solitary males. While females lead a nomadic life searching for prey in large home ranges, males are more sedentary and instead establish much smaller territories in areas with plentiful prey and access to females. The cheetah is active during the day, with peaks during dawn and dusk. It feeds on small- to medium-sized prey, mostly weighing under 40 kg (88 lb), and prefers medium-sized ungulates such as impala, springbok and Thomson\'s gazelles. The cheetah typically stalks its prey within 60–100 m (200–330 ft) before charging towards it, trips it during the chase and bites its throat to suffocate it to death. It breeds throughout the year. After a gestation of nearly three months, females give birth to a litter of three or four cubs. Cheetah cubs are highly vulnerable to predation by other large carnivores. They are weaned at around four months and are independent by around 20 months of age.\nThe cheetah is threatened by habitat loss, conflict with humans, poaching and high susceptibility to diseases. In 2016, the global cheetah population was estimated at 7,100 individuals in the wild; it is listed as Vulnerable on the IUCN Red List. It has been widely depicted in art, literature, advertising, and animation. It was tamed in ancient Egypt and trained for hunting ungulates in the Arabian Peninsula and India. It has been kept in zoos since the early 19th century.', 'source': 'https://en.wikipedia.org/wiki/Cheetah'}
```

```python
print(result["answer"])
```

```output
猎豹能够以每小时93到104公里（58到65英里）的速度奔跑。它们已经进化出了专门的速度适应性，包括轻盈的体型、细长的腿和长长的尾巴。
```

LangSmith trace: https://smith.langchain.com/public/0472c5d1-49dc-4c1c-8100-61910067d7ed/r

## 调用函数

如果你选择的LLM实现了[工具调用](/docs/concepts#functiontool-calling)功能，你可以使用它来让模型在生成答案时指定引用的提供文档。LangChain的工具调用模型实现了`.with_structured_output`方法，它将强制生成符合所需模式的答案（例如[这里](https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html#langchain_openai.chat_models.base.ChatOpenAI.with_structured_output)）。

### 引用文档

要使用标识符引用文档，我们将标识符格式化到提示中，然后使用`.with_structured_output`来强制LLM在输出中引用这些标识符。

首先，我们为输出定义一个模式。`.with_structured_output`支持多种格式，包括JSON模式和Pydantic。这里我们将使用Pydantic：

```python
from langchain_core.pydantic_v1 import BaseModel, Field
class CitedAnswer(BaseModel):
    """基于给定来源回答用户问题，并引用所使用的来源。"""
    answer: str = Field(
        ...,
        description="基于给定来源回答用户问题的答案。",
    )
    citations: List[int] = Field(
        ...,
        description="证明答案的具体来源的整数ID。",
    )
```

让我们看看当我们传入我们的函数和用户输入时，模型的输出是什么样子的：

```python
structured_llm = llm.with_structured_output(CitedAnswer)
example_q = """Brian的身高是多少？
来源：1
信息：Suzy身高6'2"
来源：2
信息：Jeremiah是金发
来源：3
信息：Brian比Suzy矮3英寸"""
result = structured_llm.invoke(example_q)
result
```

```output
CitedAnswer(answer="Brian的身高是5'11\"。", citations=[1, 3])
```

或者作为字典：

```python
result.dict()
```

```output
{'answer': "Brian的身高是5'11\"。", 'citations': [1, 3]}
```

现在我们将源标识符结构化到提示中，以便复制我们的链。我们将进行三个更改：

1. 更新提示以包括源标识符；

2. 使用`structured_llm`（即`llm.with_structured_output(CitedAnswer))；

3. 删除`StrOutputParser`，以保留输出中的Pydantic对象。

```python
def format_docs_with_id(docs: List[Document]) -> str:
    formatted = [
        f"来源ID：{i}\n文章标题：{doc.metadata['title']}\n文章摘要：{doc.page_content}"
        for i, doc in enumerate(docs)
    ]
    return "\n\n" + "\n\n".join(formatted)
rag_chain_from_docs = (
    RunnablePassthrough.assign(context=(lambda x: format_docs_with_id(x["context"])))
    | prompt
    | structured_llm
)
retrieve_docs = (lambda x: x["input"]) | retriever
chain = RunnablePassthrough.assign(context=retrieve_docs).assign(
    answer=rag_chain_from_docs
)
```

```python
result = chain.invoke({"input": "猎豹的奔跑速度有多快？"})
```

```python
print(result["answer"])
```

```output
answer='猎豹的奔跑速度为每小时93到104公里（58到65英里）。它们被称为最快的陆地动物。' citations=[0]
```

我们可以检查模型引用的索引为0的文档：

```python
print(result["context"][0])
```

```output
```markdown

The cheetah (Acinonyx jubatus) is a large cat and the fastest land animal. It has a tawny to creamy white or pale buff fur that is marked with evenly spaced, solid black spots. The head is small and rounded, with a short snout and black tear-like facial streaks. It reaches 67–94 cm (26–37 in) at the shoulder, and the head-and-body length is between 1.1 and 1.5 m (3 ft 7 in and 4 ft 11 in). Adults weigh between 21 and 72 kg (46 and 159 lb). The cheetah is capable of running at 93 to 104 km/h (58 to 65 mph); it has evolved specialized adaptations for speed, including a light build, long thin legs and a long tail.

```
猎豹（Acinonyx jubatus）是一种大型猫科动物，也是地球上奔跑速度最快的动物。它的皮毛呈黄褐色至奶油白色或淡棕色，上面布满均匀分布的黑色斑点。头部小而圆，嘴短，脸上有黑色的泪痕状条纹。猎豹肩高67-94厘米（26-37英寸），头体长1.1至1.5米（3英尺7英寸至4英尺11英寸）。成年猎豹体重在21至72公斤（46至159磅）之间。猎豹的奔跑速度可达93至104公里/小时（58至65英里/小时）；它进化出了专门的速度适应性特征，包括轻盈的体型、细长的腿和长长的尾巴。
![Cheetah](https://en.wikipedia.org/wiki/Cheetah)
```markdown

The cheetah was first described in the late 18th century. Four subspecies are recognised today that are native to Africa and central Iran. An African subspecies was introduced to India in 2022. It is now distributed mainly in small, fragmented populations in northwestern, eastern and southern Africa and central Iran. It lives in a variety of habitats such as savannahs in the Serengeti, arid mountain ranges in the Sahara, and hilly desert terrain.

```
猎豹首次被描述是在18世纪末。目前认可的四个亚种原产于非洲和伊朗中部。非洲亚种于2022年被引入印度。目前主要分布在非洲西北部、东部和南部以及伊朗中部的小而分散的种群中。它栖息在各种栖息地，如塞伦盖蒂的稀树草原、撒哈拉沙漠的干旱山脉和多丘陵的沙漠地带。
```markdown

The cheetah lives in three main social groups: females and their cubs, male "coalitions", and solitary males. While females lead a nomadic life searching for prey in large home ranges, males are more sedentary and instead establish much smaller territories in areas with plentiful prey and access to females. The cheetah is active during the day, with peaks during dawn and dusk. It feeds on small- to medium-sized prey, mostly weighing under 40 kg (88 lb), and prefers medium-sized ungulates such as impala, springbok and Thomson's gazelles. The cheetah typically stalks its prey within 60–100 m (200–330 ft) before charging towards it, trips it during the chase and bites its throat to suffocate it to death. It breeds throughout the year. After a gestation of nearly three months, females give birth to a litter of three or four cubs. Cheetah cubs are highly vulnerable to predation by other large carnivores. They are weaned at around four months and are independent by around 20 months of age.

```
猎豹生活在三个主要的社会群体中：雌性和它们的幼崽、雄性“联盟”和独居雄性。雌性过着游牧的生活，通过大范围的领地寻找猎物，而雄性更加久坐，建立在猎物丰富且有雌性出入的地区更小的领地。猎豹在白天活动，黎明和黄昏时分活动最为频繁。它以重40公斤（88磅）以下的小型至中型猎物为食，更偏好斑羚、羚羊和汤姆森氏瞪羚等中型有蹄动物。猎豹通常在距离猎物60-100米（200-330英尺）的地方悄悄接近，然后扑向猎物，追逐中将其绊倒，并咬住喉咙使其窒息而死。它全年繁殖。雌性怀孕近三个月后，会生下三到四只幼崽。猎豹幼崽极易受到其他大型食肉动物的捕食。它们在大约四个月大时断奶，到20个月大时就能独立生活。
```markdown

The cheetah is threatened by habitat loss, conflict with humans, poaching and high susceptibility to diseases. In 2016, the global cheetah population was estimated at 7,100 individuals in the wild; it is listed as Vulnerable on the IUCN Red List. It has been widely depicted in art, literature, advertising, and animation. It was tamed in ancient Egypt and trained for hunting ungulates in the Arabian Peninsula and India. It has been kept in zoos since the early 19th century.

```
猎豹受到栖息地丧失、与人类的冲突、偷猎以及对疾病的高易感性的威胁。2016年，全球野生猎豹种群估计为7100只；它被列入国际自然保护联盟红色名录的易危物种。猎豹在艺术、文学、广告和动画中被广泛描绘。它在古埃及被驯化，并被用来在阿拉伯半岛和印度狩猎有蹄动物。自19世纪初以来，猎豹一直被保存在动物园中。
![Cheetah](https://en.wikipedia.org/wiki/Cheetah)
```

```python
class Citation(BaseModel):
    source_id: int = Field(
        ...,
        description="用来证明答案的具体来源的整数ID。",
    )
    quote: str = Field(
        ...,
        description="来自指定来源的原文引用，用来证明答案。",
    )
class QuotedAnswer(BaseModel):
    """基于给定来源回答用户问题，并引用所使用的来源。"""
    answer: str = Field(
        ...,
        description="基于给定来源回答用户问题的答案。",
    )
    citations: List[Citation] = Field(
        ..., description="证明答案的来自给定来源的引用。"
    )
```

```python
rag_chain_from_docs = (
    RunnablePassthrough.assign(context=(lambda x: format_docs_with_id(x["context"])))
    | prompt
    | llm.with_structured_output(QuotedAnswer)
)
retrieve_docs = (lambda x: x["input"]) | retriever
chain = RunnablePassthrough.assign(context=retrieve_docs).assign(
    answer=rag_chain_from_docs
)
```

```python
result = chain.invoke({"input": "猎豹有多快？"})
```

这里我们看到模型从来源0中提取了相关的文本片段：

```python
result["answer"]
```

```output
QuotedAnswer(answer='猎豹的奔跑速度为每小时93到104公里（58到65英里）。', citations=[Citation(source_id=0, quote='猎豹能够以每小时93到104公里（58到65英里）的速度奔跑；它已经进化出了专门用于奔跑的适应性特征，包括轻盈的体型、修长的腿和长长的尾巴。')])
```

LangSmith trace: https://smith.langchain.com/public/0f638cc9-8409-4a53-9010-86ac28144129/r

## 直接提示

许多模型不支持函数调用。我们可以通过直接提示来实现类似的结果。让我们尝试指示模型生成结构化的XML输出：

```python
xml_system = """你是一个乐于助人的AI助手。给定用户问题和一些维基百科文章片段，回答用户问题并提供引用。如果没有一篇文章回答了问题，就说你不知道。
记住，你必须返回答案和引用。引用包括证明答案的原文引用和引用文章的ID。对于所有证明答案的引用，都要返回一个引用。使用以下格式返回最终输出：
<cited_answer>
    <answer></answer>
    <citations>
        <citation><source_id></source_id><quote></quote></citation>
        <citation><source_id></source_id><quote></quote></citation>
        ...
    </citations>
</cited_answer>
以下是维基百科文章：{context}"""
xml_prompt = ChatPromptTemplate.from_messages(
    [("system", xml_system), ("human", "{input}")]
)
```

我们现在对我们的链做类似的小更新：

1. 我们更新格式化函数，将检索到的上下文包装在XML标记中；

2. 我们不使用`.with_structured_output`（例如，因为它对于某些模型不存在）；

3. 我们使用[XMLOutputParser](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.xml.XMLOutputParser.html)来解析答案为字典。

```python
from langchain_core.output_parsers import XMLOutputParser
def format_docs_xml(docs: List[Document]) -> str:
    formatted = []
    for i, doc in enumerate(docs):
        doc_str = f"""\
    <source id=\"{i}\">
        <title>{doc.metadata['title']}</title>
        <article_snippet>{doc.page_content}</article_snippet>
    </source>"""
        formatted.append(doc_str)
    return "\n\n<sources>" + "\n".join(formatted) + "</sources>"
rag_chain_from_docs = (
    RunnablePassthrough.assign(context=(lambda x: format_docs_xml(x["context"])))
    | xml_prompt
    | llm
    | XMLOutputParser()
)
retrieve_docs = (lambda x: x["input"]) | retriever
chain = RunnablePassthrough.assign(context=retrieve_docs).assign(
    answer=rag_chain_from_docs
)
```

```python
result = chain.invoke({"input": "猎豹有多快？"})
```

注意，引用再次结构化到了答案中：

```python
result["answer"]
```

```output
{'cited_answer': [{'answer': '猎豹能够以每小时93到104公里（58到65英里）的速度奔跑。'},
  {'citations': [{'citation': [{'source_id': '0'},
      {'quote': '猎豹能够以每小时93到104公里（58到65英里）的速度奔跑；它已经进化出了专门用于奔跑的适应性特征，包括轻盈的体型、修长的腿和长长的尾巴。'}]}]}]}
```

LangSmith trace: https://smith.langchain.com/public/a3636c70-39c6-4c8f-bc83-1c7a174c237e/r

## 检索后处理

另一种方法是对我们检索到的文档进行后处理，以压缩内容，使得源内容已经足够简化，不需要模型引用特定来源或段落。例如，我们可以将每个文档分成一两个句子，嵌入它们并只保留最相关的内容。LangChain内置了一些用于此目的的组件。在这里，我们将使用一个[RecursiveCharacterTextSplitter](https://api.python.langchain.com/en/latest/text_splitter/langchain_text_splitters.RecursiveCharacterTextSplitter.html#langchain_text_splitters.RecursiveCharacterTextSplitter)，它通过在分隔子字符串上进行分割来创建指定大小的块，以及一个[EmbeddingsFilter](https://api.python.langchain.com/en/latest/retrievers/langchain.retrievers.document_compressors.embeddings_filter.EmbeddingsFilter.html#langchain.retrievers.document_compressors.embeddings_filter.EmbeddingsFilter)，它只保留具有最相关嵌入的文本。

这种方法有效地用更新后的检索器替换了我们的原始检索器，用于压缩文档。首先，我们构建检索器：

```python
from langchain.retrievers.document_compressors import EmbeddingsFilter
from langchain_core.runnables import RunnableParallel
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
splitter = RecursiveCharacterTextSplitter(
    chunk_size=400,
    chunk_overlap=0,
    separators=["\n\n", "\n", ".", " "],
    keep_separator=False,
)
compressor = EmbeddingsFilter(embeddings=OpenAIEmbeddings(), k=10)
def split_and_filter(input) -> List[Document]:
    docs = input["docs"]
    question = input["question"]
    split_docs = splitter.split_documents(docs)
    stateful_docs = compressor.compress_documents(split_docs, question)
    return [stateful_doc for stateful_doc in stateful_docs]
new_retriever = (
    RunnableParallel(question=RunnablePassthrough(), docs=retriever) | split_and_filter
)
docs = new_retriever.invoke("How fast are cheetahs?")
for doc in docs:
    print(doc.page_content)
    print("\n\n")
```

```output
成年斑鬣猫的体重在21至72千克（46至159磅）之间。斑鬣猫能够以93至104公里/小时（58至65英里/小时）的速度奔跑；它已经进化出了专门用于速度的适应性特征，包括轻盈的体型、修长的腿和长尾。
斑鬣猫（Acinonyx jubatus）是一种大型猫科动物，也是地球上奔跑速度最快的动物。它的皮毛呈黄褐色至乳白色或浅棕色，上面布满均匀分布的实心黑色斑点。头部小而圆，有短吻和黑色的泪痕状面纹。肩高67-94厘米（26-37英寸），头体长1.1-1.5米（3英尺7英寸至4英尺11英寸）。
它的奔跑速度是每秒93-104公里（58-65英里），或每秒171个身长。斑鬣猫是地球上奔跑速度最快的哺乳动物，但它的速度仅为每秒16个身长，而安娜蜂鸟是脊椎动物中已知的具有最高长度特定速度的物种。
它以小到中型猎物为食，大多重在40千克（88磅）以下，并偏爱中型有蹄类动物，如黑斑羚、斑羚和汤姆森瞪羚。斑鬣猫通常在60-100米（200-330英尺）内悄悄接近猎物，然后向其冲去，追逐过程中将其绊倒，并咬住喉咙使其窒息致死。它全年都会繁殖。
斑鬣猫最早是在18世纪末被描述的。如今认可的有四个亚种，它们分布在非洲和中部伊朗。非洲亚种于2022年被引入印度。如今，它主要分布在非洲西北部、东部和南部以及中部伊朗的小片断种群中。
斑鬣猫生活在三个主要的社会群体中：母亲和它们的幼崽、雄性“联盟”和独居的雄性。母亲们过着游牧的生活，搜索大范围的猎物活动范围，而雄性更加宅在家里，建立在有丰富猎物和雌性动物的地方的小得多的领地。斑鬣猫在白天活动，黎明和黄昏时分活动最频繁。
东南非斑鬣猫（Acinonyx jubatus jubatus）是指生活在东部和南部非洲的斑鬣猫亚种。南部非洲斑鬣猫主要生活在卡拉哈里的低地地区和沙漠、奥卡万戈三角洲的稀树草原以及南非特兰斯瓦尔地区的草原上。在纳米比亚，斑鬣猫主要分布在农田中。
亚种被称为“南非斑鬣猫”和“纳米比亚斑鬣猫”。
在印度，四只属于该亚种的斑鬣猫目前生活在马德哈·普拉德甘普邦的库诺国家公园。
Acinonyx jubatus velox是根据1913年埃德蒙德·海勒在肯尼亚高原上射杀的一只斑鬣猫提出的。
Acinonyx rex是根据1927年雷金纳德·因内斯·波科克在罗得西亚乌姆武克韦山脉的一份标本提出的。
```

接下来，我们像以前一样将其组装成我们的链：

```python
rag_chain_from_docs = (
    RunnablePassthrough.assign(context=(lambda x: format_docs(x["context"])))
    | prompt
    | llm
    | StrOutputParser()
)
chain = RunnablePassthrough.assign(
    context=(lambda x: x["input"]) | new_retriever
).assign(answer=rag_chain_from_docs)
```

```python
result = chain.invoke({"input": "How fast are cheetahs?"})
print(result["answer"])
```

```output
猎豹能够以每小时93到104公里（58到65英里）的速度奔跑，使其成为陆地上最快的动物。
```

请注意，文档内容现在已经被压缩，尽管文档对象在其元数据的“摘要”键中保留了原始内容。这些摘要不会传递给模型；只有压缩内容被传递。

```python
result["context"][0].page_content  # 传递给模型
```

```output
'成年猎豹的体重在21到72公斤（46到159磅）之间。猎豹能够以每小时93到104公里（58到65英里）的速度奔跑；它已经进化出了专门的速度适应性，包括轻盈的体型、细长的腿和长长的尾巴'
```

```python
result["context"][0].metadata["summary"]  # 原始文档
```

```output
'猎豹（Acinonyx jubatus）是一种大型猫科动物，也是陆地上最快的动物。它的皮毛呈黄褐色至乳白色或淡棕色，上面布满均匀间隔的实心黑色斑点。头部小而圆，有短吻和黑色的泪痕状面纹。肩高67-94厘米（26-37英寸），头体长1.1-1.5米（3英尺7英寸至4英尺11英寸）。成年猎豹的体重在21到72公斤（46到159磅）之间。猎豹能够以每小时93到104公里（58到65英里）的速度奔跑；它已经进化出了专门的速度适应性，包括轻盈的体型、细长的腿和长长的尾巴。\n猎豹最早是在18世纪晚期被描述的。如今认可的有四个亚种，分布在非洲和伊朗中部。非洲亚种于2022年被引入印度。目前，猎豹主要分布在非洲西北部、东部和南部以及伊朗中部的小片断种群中。它生活在各种栖息地，如塞伦盖蒂的稀树草原、撒哈拉的干旱山脉和多丘陵的沙漠地带。\n猎豹生活在三个主要的社会群体中：母猎豹和它们的幼崽、雄性“联盟”和独居的雄性。雌性猎豹过着游牧的生活，搜索大范围的猎物，而雄性更加宅居，建立在猎物丰富且能接触到雌性的地区中更小的领地。猎豹白天活动，黎明和黄昏时分活动最频繁。它以小到中型的猎物为食，大多重40公斤（88磅）以下，并且更偏好中型有蹄类动物，如黑斑羚、斑羚和汤姆森氏瞪羚。猎豹通常在60-100米（200-330英尺）内悄悄接近猎物，然后向其冲去，在追逐过程中将其绊倒，并咬住它的喉咙使其窒息而死。它全年繁殖。怀孕近三个月后，雌性生下三到四只幼崽。猎豹幼崽极易受到其他大型食肉动物的捕食。它们在大约四个月大时断奶，到大约20个月大时独立。\n猎豹受到栖息地丧失、与人类的冲突、偷猎和高易感性疾病的威胁。2016年，全球猎豹种群在野外估计为7100只；它被列为IUCN红色名录的易危物种。它在艺术、文学、广告和动画中被广泛描绘。它在古埃及被驯化，并在阿拉伯半岛和印度被用于狩猎有蹄类动物。自19世纪初以来，它一直被保存在动物园中。'
```

LangSmith 追踪链接：[https://smith.langchain.com/public/a61304fa-e5a5-4c64-a268-b0aef1130d53/r](https://smith.langchain.com/public/a61304fa-e5a5-4c64-a268-b0aef1130d53/r)

## 生成后处理

另一种方法是对我们的模型生成进行后处理。在这个例子中，我们首先只生成一个答案，然后我们将要求模型用引文注释其自己的答案。这种方法的缺点当然是它更慢、更昂贵，因为需要进行两次模型调用。

让我们将这个方法应用到我们的初始链中。

```python
class Citation(BaseModel):
    source_id: int = Field(
        ...,
        description="证明答案的特定来源的整数ID。",
    )
    quote: str = Field(
        ...,
        description="来自指定来源的原始引文，证明答案。",
    )
class AnnotatedAnswer(BaseModel):
    """用证明答案的引文注释用户问题的答案。"""
    citations: List[Citation] = Field(
        ..., description="来自给定来源的证明答案的引文。"
    )
structured_llm = llm.with_structured_output(AnnotatedAnswer)
```

```python
from langchain_core.prompts import MessagesPlaceholder
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", "{question}"),
        MessagesPlaceholder("chat_history", optional=True),
    ]
)
answer = prompt | llm
annotation_chain = prompt | structured_llm
chain = (
    RunnableParallel(
        question=RunnablePassthrough(), docs=(lambda x: x["input"]) | retriever
    )
    .assign(context=format)
    .assign(ai_message=answer)
    .assign(
        chat_history=(lambda x: [x["ai_message"]]),
        answer=(lambda x: x["ai_message"].content),
    )
    .assign(annotations=annotation_chain)
    .pick(["answer", "docs", "annotations"])
)
```

```python
result = chain.invoke({"input": "猎豹跑得多快？"})
```

```python
print(result["answer"])
```

```output
猎豹能够以每小时93到104公里（58到65英里）的速度奔跑。它们为了速度而进化出了专门的适应性特征，如轻盈的体型、细长的腿和长尾巴，使它们成为陆地上最快的动物。
```

```python
result["annotations"]
```

```output
AnnotatedAnswer(citations=[Citation(source_id=0, quote='猎豹能够以每小时93到104公里（58到65英里）的速度奔跑；它进化出了专门的适应性特征，包括轻盈的体型、细长的腿和长尾巴。')])
```

LangSmith trace: https://smith.langchain.com/public/bf5e8856-193b-4ff2-af8d-c0f4fbd1d9cb/r

```