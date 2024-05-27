# 与 Vectara 一起聊天

# 设置

您需要一个 Vectara 账户才能与 LangChain 一起使用 Vectara。要开始，请按照以下步骤操作：

1. 如果您还没有 Vectara 账户，请[注册](https://www.vectara.com/integrations/langchain)一个。完成注册后，您将获得一个 Vectara 客户 ID。您可以通过点击 Vectara 控制台窗口右上角的您的姓名来找到您的客户 ID。

2. 在您的账户中，您可以创建一个或多个语料库。每个语料库代表一个区域，用于存储从输入文档中摄取的文本数据。要创建一个语料库，请使用**"创建语料库"**按钮。然后为您的语料库提供一个名称和描述。您还可以选择定义过滤属性并应用一些高级选项。如果您点击您创建的语料库，您可以在顶部看到它的名称和语料库 ID。

3. 接下来，您需要创建 API 密钥以访问语料库。在语料库视图中点击**"授权"**选项卡，然后点击**"创建 API 密钥"**按钮。为您的密钥取一个名称，并选择您想要的是仅查询还是查询+索引。点击"创建"，现在您有一个活跃的 API 密钥。请保密此密钥。

要与 LangChain 一起使用 Vectara，您需要这三个值：客户 ID、语料库 ID 和 API 密钥。

您可以通过以下两种方式将它们提供给 LangChain：

1. 在您的环境中包含这三个变量：`VECTARA_CUSTOMER_ID`、`VECTARA_CORPUS_ID` 和 `VECTARA_API_KEY`。

> 例如，您可以使用 os.environ 和 getpass 设置这些变量如下：

```python
import os
import getpass
os.environ["VECTARA_CUSTOMER_ID"] = getpass.getpass("Vectara 客户 ID:")
os.environ["VECTARA_CORPUS_ID"] = getpass.getpass("Vectara 语料库 ID:")
os.environ["VECTARA_API_KEY"] = getpass.getpass("Vectara API 密钥:")
```

2. 将它们添加到 Vectara vectorstore 构造函数中：

```python
vectorstore = Vectara(
                vectara_customer_id=vectara_customer_id,
                vectara_corpus_id=vectara_corpus_id,
                vectara_api_key=vectara_api_key
            )
```

```python
import os
from langchain.chains import ConversationalRetrievalChain
from langchain_community.vectorstores import Vectara
from langchain_openai import OpenAI
```

加载文档。您可以将此替换为您想要的任何类型数据的加载器

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("state_of_the_union.txt")
documents = loader.load()
```

由于我们使用 Vectara，无需对文档进行分块，因为这在 Vectara 平台后端会自动完成。我们只需使用 `from_document()` 上传从文件加载的文本，并直接将其摄取到 Vectara 中：

```python
vectara = Vectara.from_documents(documents, embedding=None)
```

现在我们可以创建一个 memory 对象，这对于跟踪输入/输出并进行对话是必要的。

```python
from langchain.memory import ConversationBufferMemory
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)
```

现在我们初始化 `ConversationalRetrievalChain`：

```python
openai_api_key = os.environ["OPENAI_API_KEY"]
llm = OpenAI(openai_api_key=openai_api_key, temperature=0)
retriever = vectara.as_retriever()
d = retriever.invoke("总统关于 Ketanji Brown Jackson 说了什么", k=2)
print(d)
```

```output
[Document(page_content='布雷耶法官，感谢您的服务。总统最重要的宪法责任之一就是提名人选担任美国最高法院的职位。4天前，我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续布雷耶法官的卓越传统。她曾是一位顶级的私人执业律师。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '29486', 'len': '97'}), Document(page_content='公民们用自己的身体阻挡坦克。从学生到退休教师，他们变成了保卫祖国的士兵。正如泽连斯基总统在他在欧洲议会的演讲中所说：“光明将战胜黑暗。”乌克兰驻美国大使今晚在这里。让我们今晚在这个议会大厅向乌克兰和全世界发出明确的信号。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '1083', 'len': '117'}), Document(page_content='总共，仅去年我们在美国创造了369,000个新的制造业工作岗位。这些工作岗位的动力来自我所遇到的人，比如来自匹兹堡的工会钢铁工人家族的乔乔·伯吉斯，他今晚与我们在一起。正如俄亥俄州参议员谢罗德·布朗所说：“是时候埋葬‘锈带’的标签了。”是时候了。\n\n但是尽管我们经济中有许多亮点，创纪录的就业增长和更高的工资，但太多的家庭仍然在努力跟上账单。通货膨胀正在剥夺他们可能感受到的收益。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '14257', 'len': '77'}), Document(page_content='这对我和吉尔、卡马拉以及你们许多人来说都是个人问题。癌症是美国的第二大死因，仅次于心脏病。上个月，我宣布了我们的计划，以加速奥巴马总统六年前要我领导的癌症登月计划。我们的目标是在未来25年内将癌症死亡率至少降低50%，将更多的癌症从死刑变为可治疗的疾病。为患者和家庭提供更多支持。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '36196', 'len': '122'}), Document(page_content='六天前，俄罗斯的弗拉基米尔·普京试图动摇自由世界的基础，以为他可以让世界屈服于他的威胁。但他严重错误估计了。他以为他可以卷土重来，而世界将屈服。相反，他遭遇了他从未想象过的坚强墙壁。他遇到了乌克兰人民。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '664', 'len': '68'}), Document(page_content='我明白。我记得当我爸爸不得不离开我们在宾夕法尼亚州斯克兰顿的家去找工作的时候。我在一个家庭中长大，如果食品价格上涨，你会感受到它。这就是为什么我担任总统后的第一件事就是努力通过《美国救援计划》。因为人们正在受苦。我们需要采取行动，我们也做到了。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '8042', 'len': '97'}), Document(page_content='他拒绝了多次外交努力。他以为西方和北约不会回应。他以为他可以在国内分裂我们。我们已经准备好了。以下是我们所做的。我们进行了广泛而仔细的准备。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '2100', 'len': '42'}), Document(page_content='他以为他可以卷土重来，而世界将屈服。相反，他遭遇了他从未想象过的坚强墙壁。他遇到了乌克兰人民。从泽连斯基总统到每一个乌克兰人，他们的无畏、勇气和决心激励着世界。公民们用自己的身体阻挡坦克。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '788', 'len': '28'}), Document(page_content='普京最近对乌克兰的袭击是有预谋且无端的。他拒绝了多次外交努力。他以为西方和北约不会回应。他以为他可以在国内分裂我们。我们已经准备好了。以下是我们所做的。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '2053', 'len': '46'}), Document(page_content='一个团结的国家议程。我们能做到这一点。\n\n我的美国同胞们——今晚，我们聚集在一个神圣的地方——我们民主的堡垒。在这个国会大厦里，一代又一代的美国人在巨大的争议中讨论重大问题，并做出伟大的事情。我们为自由而战，扩大自由，打败极权主义和恐怖主义。我们建立了世界上最强大、最自由、最繁荣的国家。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '36968', 'len': '131'})]
```

```python
bot = ConversationalRetrievalChain.from_llm(
    llm, retriever, memory=memory, verbose=False
)
```

现在我们可以与我们的新机器人进行多轮对话了：

```python
query = "总统对 Ketanji Brown Jackson 有什么评价？"
result = bot.invoke({"question": query})
```

```python
result["answer"]
```

```output
"总统表示，Ketanji Brown Jackson 是全国顶尖的法律专家和一位曾在私人执业中担任高级诉讼律师的人，她将继续 Justice Breyer 的卓越传统。"
```

```python
query = "他提到她接替了谁吗？"
result = bot.invoke({"question": query})
```

```python
result["answer"]
```

```output
"Ketanji Brown Jackson 接替了联邦最高法院的 Breyer 法官。"
```

## 传入聊天记录

在上面的例子中，我们使用了一个 Memory 对象来跟踪聊天记录。我们也可以直接传入聊天记录。为了做到这一点，我们需要初始化一个没有任何 Memory 对象的链。

```python
bot = ConversationalRetrievalChain.from_llm(
    OpenAI(temperature=0), vectara.as_retriever()
)
```

以下是一个没有聊天记录的问题示例：

```python
chat_history = []
query = "总统对 Ketanji Brown Jackson 有什么评价？"
result = bot.invoke({"question": query, "chat_history": chat_history})
```

```python
result["answer"]
```

```output
"总统表示，Ketanji Brown Jackson 是全国顶尖的法律专家和一位曾在私人执业中担任高级诉讼律师的人，她将继续 Justice Breyer 的卓越传统。"
```

以下是带有一些聊天记录的问题示例：

```python
chat_history = [(query, result["answer"])]
query = "他提到她接替了谁吗？"
result = bot.invoke({"question": query, "chat_history": chat_history})
```

```python
result["answer"]
```

```output
"Ketanji Brown Jackson 接替了联邦最高法院的 Breyer 法官。"
```

## 返回源文件

您还可以轻松地从 ConversationalRetrievalChain 返回源文件。这在您想要检查返回了哪些文件时非常有用。

```python
bot = ConversationalRetrievalChain.from_llm(
    llm, vectara.as_retriever(), return_source_documents=True
)
```

```python
chat_history = []
query = "总统对 Ketanji Brown Jackson 有什么评价？"
result = bot.invoke({"question": query, "chat_history": chat_history})
```

```python
result["source_documents"][0]
```

```output
Document(page_content='Justice Breyer, thank you for your service. One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence. A former top litigator in private practice.', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '29486', 'len': '97'})
```

## 使用 `map_reduce` 的 ConversationalRetrievalChain

LangChain 支持不同类型的方式来将文档链与 ConversationalRetrievalChain 链组合在一起。

```python
from langchain.chains import LLMChain
from langchain.chains.conversational_retrieval.prompts import CONDENSE_QUESTION_PROMPT
from langchain.chains.question_answering import load_qa_chain
```

```python
question_generator = LLMChain(llm=llm, prompt=CONDENSE_QUESTION_PROMPT)
doc_chain = load_qa_chain(llm, chain_type="map_reduce")
chain = ConversationalRetrievalChain(
    retriever=vectara.as_retriever(),
    question_generator=question_generator,
    combine_docs_chain=doc_chain,
)
```

```python
chat_history = []
query = "总统对 Ketanji Brown Jackson 有什么评价？"
result = chain({"question": query, "chat_history": chat_history})
```

```python
result["answer"]
```

```output
"总统表示，他提名了联邦上诉法院法官 Ketanji Brown Jackson，她是全国顶尖的法律专家和一位曾在私人执业中担任高级诉讼律师的人。"
```

## 使用带源文件的 Question Answering 的 ConversationalRetrievalChain

您还可以将此链与带源文件的 Question Answering 链一起使用。

```python
from langchain.chains.qa_with_sources import load_qa_with_sources_chain
```

```python
question_generator = LLMChain(llm=llm, prompt=CONDENSE_QUESTION_PROMPT)
doc_chain = load_qa_with_sources_chain(llm, chain_type="map_reduce")
chain = ConversationalRetrievalChain(
    retriever=vectara.as_retriever(),
    question_generator=question_generator,
    combine_docs_chain=doc_chain,
)
```

```python
chat_history = []
query = "总统对 Ketanji Brown Jackson 有什么评价？"
result = chain({"question": query, "chat_history": chat_history})
```

```python
result["answer"]
```

```output
" 总统表示，Ketanji Brown Jackson 是全国顶尖的法律智囊之一，曾是一名私人执业中的顶尖诉讼律师。
来源：langchain"
```

## 使用 `stdout` 进行流式传输的 ConversationalRetrievalChain

在这个示例中，链的输出将以令牌的形式流式传输到 `stdout`。

```python
from langchain.chains.conversational_retrieval.prompts import (
    CONDENSE_QUESTION_PROMPT,
    QA_PROMPT,
)
from langchain.chains.llm import LLMChain
from langchain.chains.question_answering import load_qa_chain
from langchain_core.callbacks import StreamingStdOutCallbackHandler
# 使用流式 llm 组合文档和单独的非流式 llm 生成问题构建 ConversationalRetrievalChain
llm = OpenAI(temperature=0, openai_api_key=openai_api_key)
streaming_llm = OpenAI(
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()],
    temperature=0,
    openai_api_key=openai_api_key,
)
question_generator = LLMChain(llm=llm, prompt=CONDENSE_QUESTION_PROMPT)
doc_chain = load_qa_chain(streaming_llm, chain_type="stuff", prompt=QA_PROMPT)
bot = ConversationalRetrievalChain(
    retriever=vectara.as_retriever(),
    combine_docs_chain=doc_chain,
    question_generator=question_generator,
)
```

```python
chat_history = []
query = "总统对凯坦吉·布朗·杰克逊（Ketanji Brown Jackson）有何评价"
result = bot.invoke({"question": query, "chat_history": chat_history})
```

```output
 总统表示凯坦吉·布朗·杰克逊是全国顶尖的法律专家之一，曾是一名顶尖的私人执业诉讼律师，并且她将继续布雷耶法官（Justice Breyer）卓越的传统。
```

```python
chat_history = [(query, result["answer"])]
query = "他是否提到她接替了谁"
result = bot.invoke({"question": query, "chat_history": chat_history})
```

```output
 凯坦吉·布朗·杰克逊接替了美国最高法院的布雷耶法官（Justice Breyer）。
```

## get_chat_history 函数

您还可以指定一个 `get_chat_history` 函数，用于格式化 `chat_history` 字符串。

```python
def get_chat_history(inputs) -> str:
    res = []
    for human, ai in inputs:
        res.append(f"人类:{human}\nAI:{ai}")
    return "\n".join(res)
bot = ConversationalRetrievalChain.from_llm(
    llm, vectara.as_retriever(), get_chat_history=get_chat_history
)
```

```python
chat_history = []
query = "总统对凯坦吉·布朗·杰克逊（Ketanji Brown Jackson）有何评价"
result = bot.invoke({"question": query, "chat_history": chat_history})
```

```python
result["answer"]
```

```output
"总统表示凯坦吉·布朗·杰克逊是全国顶尖的法律专家之一，曾是一名顶尖的私人执业诉讼律师，并且她将继续布雷耶法官（Justice Breyer）卓越的传统。"
```