# 如何重新排序检索到的结果，使得最相关的文档不位于中间位置

无论您的模型架构如何，当您包含10个以上的检索到的文档时，性能会显著下降。

简而言之：当模型必须在长篇文本的中间位置访问相关信息时，它们往往会忽略提供的文档。

参考：[https://arxiv.org/abs/2307.03172](https://arxiv.org/abs/2307.03172)

为了避免这个问题，您可以在检索后重新排序文档，以避免性能下降。

```python
%pip install --upgrade --quiet  sentence-transformers langchain-chroma langchain langchain-openai langchain-huggingface > /dev/null
```

```python
from langchain.chains import LLMChain, StuffDocumentsChain
from langchain_chroma import Chroma
from langchain_community.document_transformers import (
    LongContextReorder,
)
from langchain_core.prompts import PromptTemplate
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_openai import OpenAI
# 获取嵌入向量
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
texts = [
    "篮球是一项伟大的运动。",
    "《Fly me to the moon》是我最喜欢的歌曲之一。",
    "凯尔特人队是我最喜欢的球队。",
    "这是一篇关于波士顿凯尔特人队的文档。",
    "我非常喜欢去电影院。",
    "波士顿凯尔特人队以20分的优势赢得了比赛。",
    "这只是一段随机的文本。",
    "《Elden Ring》是过去15年中最好的游戏之一。",
    "L·科尔内特是凯尔特人队最好的球员之一。",
    "拉里·伯德是一位标志性的NBA球员。",
]
# 创建一个检索器
retriever = Chroma.from_texts(texts, embedding=embeddings).as_retriever(
    search_kwargs={"k": 10}
)
query = "你能告诉我关于凯尔特人队的信息吗？"
# 按相关性得分获取相关文档
docs = retriever.get_relevant_documents(query)
docs
```

```output
[Document(page_content='这是一篇关于波士顿凯尔特人队的文档。'),
 Document(page_content='凯尔特人队是我最喜欢的球队。'),
 Document(page_content='L·科尔内特是凯尔特人队最好的球员之一。'),
 Document(page_content='波士顿凯尔特人队以20分的优势赢得了比赛。'),
 Document(page_content='拉里·伯德是一位标志性的NBA球员。'),
 Document(page_content='《Elden Ring》是过去15年中最好的游戏之一。'),
 Document(page_content='篮球是一项伟大的运动。'),
 Document(page_content='我非常喜欢去电影院。'),
 Document(page_content='《Fly me to the moon》是我最喜欢的歌曲之一。'),
 Document(page_content='这只是一段随机的文本。')]
```

```python
# 重新排序文档：
# 不太相关的文档将位于列表的中间位置，而更相关的文档将位于开头/结尾。
reordering = LongContextReorder()
reordered_docs = reordering.transform_documents(docs)
# 确认4个相关文档位于开头和结尾。
reordered_docs
```

```output
[Document(page_content='凯尔特人队是我最喜欢的球队。'),
 Document(page_content='波士顿凯尔特人队以20分的优势赢得了比赛。'),
 Document(page_content='《Elden Ring》是过去15年中最好的游戏之一。'),
 Document(page_content='我非常喜欢去电影院。'),
 Document(page_content='这只是一段随机的文本。'),
 Document(page_content='《Fly me to the moon》是我最喜欢的歌曲之一。'),
 Document(page_content='篮球是一项伟大的运动。'),
 Document(page_content='拉里·伯德是一位标志性的NBA球员。'),
 Document(page_content='L·科尔内特是凯尔特人队最好的球员之一。'),
 Document(page_content='这是一篇关于波士顿凯尔特人队的文档。')]
```

```python
# 我们准备并运行一个自定义的Stuff链，将重新排序的文档作为上下文。
# 覆盖提示
document_prompt = PromptTemplate(
    input_variables=["page_content"], template="{page_content}"
)
document_variable_name = "context"
llm = OpenAI()
stuff_prompt_override = """给定以下文本摘录：
-----
{context}
-----
请回答以下问题：
{query}"""
prompt = PromptTemplate(
    template=stuff_prompt_override, input_variables=["context", "query"]
)
# 实例化链
llm_chain = LLMChain(llm=llm, prompt=prompt)
chain = StuffDocumentsChain(
    llm_chain=llm_chain,
    document_prompt=document_prompt,
    document_variable_name=document_variable_name,
)
chain.run(input_documents=reordered_docs, query=query)
```

```output
'\n\n这些文本摘录中有四个提到了凯尔特人队。它们被提到为作者最喜欢的球队、一场篮球比赛的赢家、一支拥有最好球员的队伍以及一支拥有特定球员的队伍。此外，最后一个摘录表明该文档是关于波士顿凯尔特人队的。这表明凯尔特人队是一支篮球队，可能来自波士顿，是一支有名且在过去有过成功球员和比赛的队伍。 '
```