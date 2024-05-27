

# AI21SemanticTextSplitter

这个示例演示了如何在 LangChain 中使用 AI21SemanticTextSplitter。

## 安装

```python
pip install langchain-ai21
```

## 环境设置

我们需要获取 AI21 API 密钥并设置 AI21_API_KEY 环境变量：

```python
import os
from getpass import getpass
os.environ["AI21_API_KEY"] = getpass()
```

## 示例用法

### 按语义意义拆分文本

这个示例展示了如何使用 AI21SemanticTextSplitter 根据语义意义将文本拆分成块。

```python
from langchain_ai21 import AI21SemanticTextSplitter
TEXT = (
    "我们都经历过阅读冗长、乏味的文本 - 财务报告、法律文件或条款和条件（尽管，谁真的会读那些条款和条件呢？）。\n"
    "想象一家雇佣数十万名员工的公司。在当今信息过载的时代，将近30%的工作时间用于处理文件。毫不奇怪，"
    "一些文件故意设计得又长又复杂（你知道吗，阅读所有的隐私政策几乎要花去四分之一年的时间？）。除了效率低下，"
    "员工可能会简单地不去阅读一些文件（例如，只有16%的员工在签署雇佣合同之前会完整阅读！）。\n"
    "这就是 AI 驱动的摘要工具可以帮助的地方：用户可以（理想情况下）从文本中快速提取相关信息，而不是阅读整个文件，"
    "这是繁琐且耗时的。借助大型语言模型，开发这些工具比以往任何时候都更容易，您可以为用户提供根据其偏好定制的摘要。\n"
    "大型语言模型自然会遵循输入（提示）中的模式，并提供与之相同模式的连贯完成。为此，我们希望在输入中提供几个示例（“few-shot prompt”），"
    "以便它们可以跟进。为您的问题创建正确提示的过程称为提示工程，您可以在这里阅读更多相关信息。"
)
semantic_text_splitter = AI21SemanticTextSplitter()
chunks = semantic_text_splitter.split_text(TEXT)
print(f"文本已被拆分为 {len(chunks)} 个块。")
for chunk in chunks:
    print(chunk)
    print("====")
```

### 按语义意义拆分文本并合并

这个示例展示了如何使用 AI21SemanticTextSplitter 根据语义意义将文本拆分成块，然后根据 `chunk_size` 合并这些块。

```python
from langchain_ai21 import AI21SemanticTextSplitter
TEXT = (
    "我们都经历过阅读冗长、乏味的文本 - 财务报告、法律文件或条款和条件（尽管，谁真的会读那些条款和条件呢？）。\n"
    "想象一家雇佣数十万名员工的公司。在当今信息过载的时代，将近30%的工作时间用于处理文件。毫不奇怪，"
    "一些文件故意设计得又长又复杂（你知道吗，阅读所有的隐私政策几乎要花去四分之一年的时间？）。除了效率低下，"
    "员工可能会简单地不去阅读一些文件（例如，只有16%的员工在签署雇佣合同之前会完整阅读！）。\n"
    "这就是 AI 驱动的摘要工具可以帮助的地方：用户可以（理想情况下）从文本中快速提取相关信息，而不是阅读整个文件，"
    "这是繁琐且耗时的。借助大型语言模型，开发这些工具比以往任何时候都更容易，您可以为用户提供根据其偏好定制的摘要。\n"
    "大型语言模型自然会遵循输入（提示）中的模式，并提供与之相同模式的连贯完成。为此，我们希望在输入中提供几个示例（“few-shot prompt”），"
    "以便它们可以跟进。为您的问题创建正确提示的过程称为提示工程，您可以在这里阅读更多相关信息。"
)
semantic_text_splitter_chunks = AI21SemanticTextSplitter(chunk_size=1000)
chunks = semantic_text_splitter_chunks.split_text(TEXT)
print(f"文本已被拆分为 {len(chunks)} 个块。")
for chunk in chunks:
    print(chunk)
    print("====")
```

### 将文本拆分为文档

这个示例展示了如何使用 AI21SemanticTextSplitter 根据语义意义将文本拆分为基于语义意义的文档。元数据将包含每个文档的类型。

```python
from langchain_ai21 import AI21SemanticTextSplitter
TEXT = (
    "我们都经历过阅读冗长、乏味的文本 - 财务报告、法律文件或条款和条件（尽管，谁真的会读那些条款和条件呢？）。\n"
    "想象一家雇佣数十万名员工的公司。在当今信息过载的时代，将近30%的工作时间用于处理文件。毫不奇怪，"
    "一些文件故意设计得又长又复杂（你知道吗，阅读所有的隐私政策几乎要花去四分之一年的时间？）。除了效率低下，"
    "员工可能会简单地不去阅读一些文件（例如，只有16%的员工在签署雇佣合同之前会完整阅读！）。\n"
    "这就是 AI 驱动的摘要工具可以帮助的地方：用户可以（理想情况下）从文本中快速提取相关信息，而不是阅读整个文件，"
    "这是繁琐且耗时的。借助大型语言模型，开发这些工具比以往任何时候都更容易，您可以为用户提供根据其偏好定制的摘要。\n"
    "大型语言模型自然会遵循输入（提示）中的模式，并提供与之相同模式的连贯完成。为此，我们希望在输入中提供几个示例（“few-shot prompt”），"
    "以便它们可以跟进。为您的问题创建正确提示的过程称为提示工程，您可以在这里阅读更多相关信息。"
)
semantic_text_splitter = AI21SemanticTextSplitter()
documents = semantic_text_splitter.split_text_to_documents(TEXT)
print(f"文本已被拆分为 {len(documents)} 个文档。")
for doc in documents:
    print(f"type: {doc.metadata['source_type']}")
    print(f"text: {doc.page_content}")
    print("====")
### 利用元数据创建文档
这个例子展示了如何使用 AI21SemanticTextSplitter 从文本中创建文档，并为每个文档添加自定义元数据。
```python

from langchain_ai21 import AI21SemanticTextSplitter

TEXT = (

    "我们都经历过阅读长篇、乏味、无聊的文字的经历 - 财务报告、法律文件，或者条款和条件（尽管，说实话，谁真的会读那些条款和条件呢？）。\n"

    "想象一家雇佣了数十万名员工的公司。在当今信息过载的时代，将近30%的工作日都花在处理文件上。毫不奇怪，因为其中一些文件故意写得又长又复杂（你知道吗，阅读所有的隐私政策几乎要花去四分之一年的时间？）。除了低效外，员工可能会简单地不去阅读一些文件（例如，只有16%的员工在签署雇佣合同前会完整阅读！）。\n这就是 AI 驱动的摘要工具可以帮助的地方：用户可以（理想情况下）快速从文本中提取相关信息，而不是阅读整个文件，这是件乏味且耗时的事情。借助大型语言模型，开发这些工具比以往任何时候都更容易，你可以为用户提供一个特别适合他们偏好的摘要。\n大型语言模型自然会遵循输入（提示）中的模式，并提供与相同模式相符的连贯完成。因此，我们希望在输入中提供几个示例（“few-shot prompt”），以便它们可以跟进。为您的问题创建正确的提示的过程称为提示工程，您可以在这里阅读更多相关信息。"

)

semantic_text_splitter = AI21SemanticTextSplitter()

texts = [TEXT]

documents = semantic_text_splitter.create_documents(

    texts=texts, metadatas=[{"pikachu": "pika pika"}]

)

print(f"文本已分成 {len(documents)} 个文档。")

for doc in documents:

    print(f"元数据: {doc.metadata}")

    print(f"文本: {doc.page_content}")

    print("====")

```
### 根据起始索引将文本分割为文档
这个例子展示了如何使用 AI21SemanticTextSplitter 根据语义意义将文本分割为文档。元数据将包含每个文档的起始索引。
**注意**，起始索引提供了每个文档块的顺序指示，而不是每个块的实际起始索引。
```python
from langchain_ai21 import AI21SemanticTextSplitter
TEXT = (
    "我们都经历过阅读长篇、乏味、无聊的文字的经历 - 财务报告、法律文件，或者条款和条件（尽管，说实话，谁真的会读那些条款和条件呢？）。\n"
    "想象一家雇佣了数十万名员工的公司。在当今信息过载的时代，将近30%的工作日都花在处理文件上。毫不奇怪，因为其中一些文件故意写得又长又复杂（你知道吗，阅读所有的隐私政策几乎要花去四分之一年的时间？）。除了低效外，员工可能会简单地不去阅读一些文件（例如，只有16%的员工在签署雇佣合同前会完整阅读！）。\n这就是 AI 驱动的摘要工具可以帮助的地方：用户可以（理想情况下）快速从文本中提取相关信息，而不是阅读整个文件，这是件乏味且耗时的事情。借助大型语言模型，开发这些工具比以往任何时候都更容易，你可以为用户提供一个特别适合他们偏好的摘要。\n大型语言模型自然会遵循输入（提示）中的模式，并提供与相同模式相符的连贯完成。因此，我们希望在输入中提供几个示例（“few-shot prompt”），以便它们可以跟进。为您的问题创建正确的提示的过程称为提示工程，您可以在这里阅读更多相关信息。"
)
semantic_text_splitter = AI21SemanticTextSplitter(add_start_index=True)
documents = semantic_text_splitter.create_documents(texts=[TEXT])
print(f"文本已分成 {len(documents)} 个文档。")
for doc in documents:
    print(f"起始索引: {doc.metadata['start_index']}")
    print(f"文本: {doc.page_content}")
    print("====")
```
### 文档分割
这个例子展示了如何使用 AI21SemanticTextSplitter 根据语义意义将文档列表分割为块。
```python
from langchain_ai21 import AI21SemanticTextSplitter
from langchain_core.documents import Document
TEXT = (
我们都经历过阅读冗长、乏味的文字——财务报告、法律文件或条款和条件（不过，说实话，谁真的会读那些条款和条件呢？）。
想象一下一个雇佣了数十万名员工的公司。在当今信息过载的时代，近30%的工作时间都花在处理文件上。这并不奇怪，因为其中一些文件故意设计得又长又复杂（你知道吗，阅读所有的隐私政策将花费将近四分之一年的时间？）。除了效率低下，员工可能会简单地不去阅读一些文件（例如，只有16%的员工在签署雇佣合同之前完整阅读了合同！）。
这就是AI驱动的摘要工具可以发挥作用的地方：用户可以（理想情况下）从文本中快速提取相关信息，而不是阅读整个文件，这样既节省时间又不乏味。借助大型语言模型，开发这些工具比以往任何时候都更容易，您可以为用户提供根据其偏好定制的摘要。
大型语言模型自然地遵循输入（提示）中的模式，并提供与这些模式相一致的连贯完成。为此，我们希望在输入（"few-shot prompt"）中提供几个示例，以便它们可以按照这些示例进行操作。为您的问题创建正确的提示的过程称为提示工程，您可以在这里阅读更多相关信息。
```python
semantic_text_splitter = AI21SemanticTextSplitter()
document = Document(page_content=TEXT, metadata={"hello": "goodbye"})
documents = semantic_text_splitter.split_documents([document])
print(f"文档列表已分割为{len(documents)}个文档。")
for doc in documents:
    print(f"文本：{doc.page_content}")
    print(f"元数据：{doc.metadata}")
    print("====")
```