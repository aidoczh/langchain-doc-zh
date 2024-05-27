# 证券交易委员会备案

[证券交易委员会备案](https://www.sec.gov/edgar)是提交给美国证券交易委员会（SEC）的财务报表或其他正式文件。公开公司、某些内部人员和经纪人被要求定期进行`SEC备案`。投资者和金融专业人士依赖这些备案获取他们评估投资对象公司的信息。

`SEC备案`数据由[Kay.ai](https://kay.ai)和[Cybersyn](https://www.cybersyn.com/)通过[Snowflake Marketplace](https://app.snowflake.com/marketplace/providers/GZTSZAS2KCS/Cybersyn%2C%20Inc)提供支持。

## 设置

首先，您需要安装`kay`软件包。您还需要一个API密钥：您可以在[https://kay.ai](https://kay.ai/)免费获取一个。一旦您获得了API密钥，您必须将其设置为环境变量`KAY_API_KEY`。

在这个示例中，我们将使用`KayAiRetriever`。查看[kay笔记本](/docs/integrations/retrievers/kay)以获取有关它接受的参数的更详细信息。

```python
# 为Kay和OpenAI设置API密钥
from getpass import getpass
KAY_API_KEY = getpass()
OPENAI_API_KEY = getpass()
```

```output
 ········
 ········
```

```python
import os
os.environ["KAY_API_KEY"] = KAY_API_KEY
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

## 示例

```python
from langchain.chains import ConversationalRetrievalChain
from langchain_community.retrievers import KayAiRetriever
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt-3.5-turbo")
retriever = KayAiRetriever.create(
    dataset_id="company", data_types=["10-K", "10-Q"], num_contexts=6
)
qa = ConversationalRetrievalChain.from_llm(model, retriever=retriever)
```

```python
questions = [
    "Nvidia过去三个季度的支出模式是什么？",
    # "可再生能源部门最近面临的一些挑战是什么？",
]
chat_history = []
for question in questions:
    result = qa({"question": question, "chat_history": chat_history})
    chat_history.append((question, result["answer"]))
    print(f"-> **问题**: {question} \n")
    print(f"**答案**: {result['answer']} \n")
```

```output
-> **问题**: Nvidia过去三个季度的支出模式是什么？ 
**答案**: 根据提供的信息，以下是NVIDIA过去三个季度支出的模式：
1. 研发支出：
   - 2022年第三季度：比2021年第三季度增长了34%。
   - 2023年第一季度：比2022年第一季度增长了40%。
   - 2022年第二季度：比2021年第二季度增长了25%。
   总体而言，研发支出在过去三个季度持续增加。
2. 销售、总务和管理支出：
   - 2022年第三季度：比2021年第三季度增长了8%。
   - 2023年第一季度：比2022年第一季度增长了14%。
   - 2022年第二季度：比2021年第二季度减少了16%。
   销售、总务和管理支出的模式不太一致，一些季度显示增长，而另一些季度显示下降。
3. 总营运支出：
   - 2022年第三季度：比2021年第三季度增长了25%。
   - 2023年第一季度：比2022年第一季度增长了113%。
   - 2022年第二季度：比2021年第二季度增长了9%。
   总营运支出在过去三个季度一般上升，2023年第一季度有显著增长。
总体而言，模式表明研发支出和总营运支出持续增加，而销售、总务和管理支出有些波动。
```