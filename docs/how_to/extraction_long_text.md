# 如何处理提取时的长文本

在处理文件（如PDF）时，你可能会遇到超过语言模型上下文窗口的文本。为了处理这些文本，可以考虑以下策略：

1. **更换语言模型** 选择支持更大上下文窗口的语言模型。

2. **蛮力法** 将文档分块，从每个块中提取内容。

3. **RAG** 将文档分块，对块进行索引，并仅从“相关”的块中提取内容。

请记住，这些策略有不同的权衡，最佳策略可能取决于你设计的应用程序！

本指南演示了如何实现策略2和3。

## 设置

我们需要一些示例数据！让我们下载一篇关于[汽车的维基百科文章](https://en.wikipedia.org/wiki/Car)，并将其作为LangChain [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html)加载。

```python
import re
import requests
from langchain_community.document_loaders import BSHTMLLoader
# 下载内容
response = requests.get("https://en.wikipedia.org/wiki/Car")
# 将其写入文件
with open("car.html", "w", encoding="utf-8") as f:
    f.write(response.text)
# 使用HTML解析器加载
loader = BSHTMLLoader("car.html")
document = loader.load()[0]
# 清理代码
# 将连续的换行符替换为单个换行符
document.page_content = re.sub("\n\n+", "\n", document.page_content)
```

```python
print(len(document.page_content))
```

```output
79174
```

## 定义模式

按照[提取教程](/docs/tutorials/extraction)的要求，我们将使用Pydantic来定义我们希望提取的信息的模式。在这种情况下，我们将提取一个包含年份和描述的“关键发展”（例如重要历史事件）的列表。

请注意，我们还包括一个`evidence`键，并指示模型以逐字的方式提供与文章中提取的年份和描述信息相关的句子。这样可以将提取结果与（模型对）原始文档中的文本进行比较。

```python
from typing import List, Optional
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.pydantic_v1 import BaseModel, Field
class KeyDevelopment(BaseModel):
    """汽车历史发展的信息。"""
    year: int = Field(
        ..., description="发生重要历史发展的年份。"
    )
    description: str = Field(
        ..., description="这一年发生了什么？有什么发展？"
    )
    evidence: str = Field(
        ...,
        description="重复提取年份和描述信息的句子（逐字）",
    )
class ExtractionData(BaseModel):
    """汽车历史中的关键发展的提取信息。"""
    key_developments: List[KeyDevelopment]
# 定义一个自定义提示，提供说明和任何其他上下文。
# 1) 可以在提示模板中添加示例以提高提取质量
# 2) 引入其他参数以考虑上下文（例如，包含从中提取文本的文档的元数据）
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "你是一个在文本中识别关键历史发展的专家。只提取重要的历史发展。如果文本中没有重要信息，则不提取任何内容。",
        ),
        ("human", "{text}"),
    ]
)
```

## 创建提取器

让我们选择一个语言模型。因为我们使用了工具调用，所以我们需要一个支持工具调用功能的模型。请参阅[此表格](/docs/integrations/chat)以获取可用的语言模型。

```python
extractor = prompt | llm.with_structured_output(
    schema=ExtractionData,
    include_raw=False,
)
```

## 蛮力法

将文档分成块，使得每个块都适合语言模型的上下文窗口。

```python
from langchain_text_splitters import TokenTextSplitter
text_splitter = TokenTextSplitter(
    # 控制每个块的大小
    chunk_size=2000,
    # 控制块之间的重叠
    chunk_overlap=20,
)
texts = text_splitter.split_text(document.page_content)
```

使用[batch](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html)功能在每个块上**并行**运行提取！

:::tip

通常可以使用`.batch()`来并行化提取！`.batch`在内部使用线程池来帮助你并行化工作负载。

如果您的模型通过 API 公开，这可能会加快您的提取流程！:::

```python
# 仅限于前3个块
# 这样可以快速重新运行代码
first_few = texts[:3]
extractions = extractor.batch(
    [{"text": text} for text in first_few],
    {"max_concurrency": 5},  # 通过传递最大并发数来限制并发！
)
```

### 合并结果

从各个块中提取数据后，我们将希望将提取结果合并在一起。

```python
key_developments = []
for extraction in extractions:
    key_developments.extend(extraction.key_developments)
key_developments[:10]
```

```output
[KeyDevelopment(year=1966, description='Toyota Corolla 开始生产，成为有史以来销量最高的汽车系列。', evidence='自 1966 年以来生产的 Toyota Corolla 是有史以来销量最高的汽车系列。'),
 KeyDevelopment(year=1769, description='Nicolas-Joseph Cugnot 制造了第一辆蒸汽动力公路车。', evidence='法国发明家 Nicolas-Joseph Cugnot 于 1769 年制造了第一辆蒸汽动力公路车。'),
 KeyDevelopment(year=1808, description='François Isaac de Rivaz 设计并制造了第一辆内燃机汽车。', evidence='瑞士发明家 François Isaac de Rivaz 于 1808 年设计并制造了第一辆内燃机汽车。'),
 KeyDevelopment(year=1886, description='Carl Benz 申请了他的 Benz Patent-Motorwagen 专利，发明了现代汽车。', evidence='现代汽车——一种实用的、适合日常使用的汽车——是在 1886 年发明的，德国发明家 Carl Benz 申请了他的 Benz Patent-Motorwagen 专利。'),
 KeyDevelopment(year=1908, description='Ford Model T，最早由大众买得起的汽车之一，开始生产。', evidence='最早由大众买得起的汽车之一是 Ford Model T，于 1908 年开始生产，由美国福特汽车公司制造。'),
 KeyDevelopment(year=1888, description='Bertha Benz 进行了第一次汽车长途旅行，以证明她丈夫的发明适合上路。', evidence='1888 年 8 月，Carl Benz 的妻子 Bertha Benz 进行了第一次汽车长途旅行，以证明她丈夫的发明适合上路。'),
 KeyDevelopment(year=1896, description='Benz 设计并申请了第一台内燃式平对置发动机，称为 boxermotor。', evidence='1896 年，Benz 设计并申请了第一台内燃式平对置发动机，称为 boxermotor。'),
 KeyDevelopment(year=1897, description='Nesselsdorfer Wagenbau 生产了 Präsident 汽车，世界上最早的工厂制造汽车之一。', evidence='中欧第一辆汽车和世界上最早的工厂制造汽车之一，是捷克公司 Nesselsdorfer Wagenbau（后更名为 Tatra）于 1897 年生产的 Präsident 汽车。'),
 KeyDevelopment(year=1890, description='Daimler Motoren Gesellschaft（DMG）由 Daimler 和 Maybach 在康斯塔特创立。', evidence='Daimler 和 Maybach 于 1890 年在康斯塔特创立了 Daimler Motoren Gesellschaft（DMG）。'),
 KeyDevelopment(year=1891, description='Auguste Doriot 和 Louis Rigoulot 驾驶 Daimler 动力的 Peugeot Type 3 完成了由汽油驱动车辆的最长旅程。', evidence='1891 年，Auguste Doriot 和他的标致同事 Louis Rigoulot 驾驶他们自行设计和建造的 Daimler 动力 Peugeot Type 3 完成了从瓦朗蒂涅到巴黎和布雷斯特再返回的 2,100 公里（1,300 英里）最长汽油驱动车辆旅程。')]
```

## 基于 RAG 的方法

另一个简单的想法是将文本分块，但不是从每个块中提取信息，而是专注于最相关的块。

:::caution

识别哪些块是相关的可能很困难。

例如，在这里使用的“汽车”文章中，大部分文章都包含关键发展信息。因此，通过使用 RAG，我们可能会丢弃很多相关信息。

我们建议尝试您的用例，并确定这种方法是否有效。

:::

要实施基于 RAG 的方法：

1. 将文档分块并对其进行索引（例如，在向量存储中）；

2. 在向量存储上添加一个检索步骤，以在提取器链之前使用向量存储。

这里有一个依赖于 `FAISS` 向量存储的简单示例。

```python
from langchain_community.vectorstores import FAISS
from langchain_core.documents import Document
from langchain_core.runnables import RunnableLambda
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
texts = text_splitter.split_text(document.page_content)
vectorstore = FAISS.from_texts(texts, embedding=OpenAIEmbeddings())
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 1}
)  # 仅从第一个文档中提取
```

在这种情况下，RAG 提取器只查看顶部文档。

```python
rag_extractor = {
    "text": retriever | (lambda docs: docs[0].page_content)  # 获取顶部文档的内容
} | extractor
```

```python
results = rag_extractor.invoke("与汽车相关的重要发展")
for key_development in results.key_developments:
    print(key_development)
```

```output
year=1869 description='玛丽·沃德成为爱尔兰帕森斯敦有记录以来的第一位车祸死亡者。' evidence='玛丽·沃德成为爱尔兰帕森斯敦有记录以来的第一位车祸死亡者，发生在1869年。'
year=1899 description='亨利·布利斯成为美国纽约市的第一位行人车祸死亡者。' evidence='亨利·布利斯成为美国纽约市有记录以来的第一位行人车祸死亡者，发生在1899年。'
year=2030 description='阿姆斯特丹将禁止所有使用化石燃料的车辆。' evidence='从2030年起，阿姆斯特丹将禁止所有使用化石燃料的车辆。'
```

## 常见问题

不同的方法在成本、速度和准确性方面各有优劣。

请注意以下问题：

- 对内容进行分块可能导致 LLM 无法提取信息，特别是当信息分布在多个块中时。

- 大块重叠可能导致相同的信息被提取两次，因此需要进行去重处理！

- LLM 可能会编造数据。如果在大段文本中寻找单个事实并采用蛮力方法，可能会得到更多虚构的数据。