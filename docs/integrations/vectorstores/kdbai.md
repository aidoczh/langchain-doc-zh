# KDB.AI

[KDB.AI](https://kdb.ai/) 是一个功能强大的基于知识的向量数据库和搜索引擎，可以通过提供先进的搜索、推荐和个性化功能，让您能够利用实时数据构建可扩展、可靠的人工智能应用程序。

[这个示例](https://github.com/KxSystems/kdbai-samples/blob/main/document_search/document_search.ipynb) 演示了如何使用 KDB.AI 对非结构化文本文档进行语义搜索。

要访问您的端点和 API 密钥，请在[KDB.AI这里注册](https://kdb.ai/get-started/)。

要设置您的开发环境，请按照[KDB.AI先决条件页面](https://code.kx.com/kdbai/pre-requisites.html)上的说明进行操作。

以下示例演示了通过 LangChain 与 KDB.AI 进行交互的一些方式。

## 导入所需的包

```python
import os
import time
from getpass import getpass
import kdbai_client as kdbai
import pandas as pd
import requests
from langchain.chains import RetrievalQA
from langchain_community.document_loaders import PyPDFLoader
from langchain_community.vectorstores import KDBAI
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
```

```python
KDBAI_ENDPOINT = input("KDB.AI 端点: ")
KDBAI_API_KEY = getpass("KDB.AI API 密钥: ")
os.environ["OPENAI_API_KEY"] = getpass("OpenAI API 密钥: ")
```

```output
KDB.AI 端点:  https://ui.qa.cld.kx.com/instance/pcnvlmi860
KDB.AI API 密钥:  ········
OpenAI API 密钥:  ········
```

```python
TEMP = 0.0
K = 3
```

## 创建 KBD.AI 会话

```python
print("创建 KDB.AI 会话...")
session = kdbai.Session(endpoint=KDBAI_ENDPOINT, api_key=KDBAI_API_KEY)
```

```output
创建 KDB.AI 会话...
```

## 创建表

```python
print('创建表 "documents"...')
schema = {
    "columns": [
        {"name": "id", "pytype": "str"},
        {"name": "text", "pytype": "bytes"},
        {
            "name": "embeddings",
            "pytype": "float32",
            "vectorIndex": {"dims": 1536, "metric": "L2", "type": "hnsw"},
        },
        {"name": "tag", "pytype": "str"},
        {"name": "title", "pytype": "bytes"},
    ]
}
table = session.create_table("documents", schema)
```

```output
创建表 "documents"...
```

```python
%%time
URL = "https://www.conseil-constitutionnel.fr/node/3850/pdf"
PDF = "Déclaration_des_droits_de_l_homme_et_du_citoyen.pdf"
open(PDF, "wb").write(requests.get(URL).content)
```

```output
CPU times: user 44.1 ms, sys: 6.04 ms, total: 50.2 ms
Wall time: 213 ms
```

```output
562978
```

## 读取 PDF

```python
%%time
print("读取 PDF...")
loader = PyPDFLoader(PDF)
pages = loader.load_and_split()
len(pages)
```

```output
读取 PDF...
CPU times: user 156 ms, sys: 12.5 ms, total: 169 ms
Wall time: 183 ms
```

```output
3
```

## 从 PDF 文本创建向量数据库

```python
%%time
print("从 PDF 文本创建向量数据库...")
embeddings = OpenAIEmbeddings(model="text-embedding-ada-002")
texts = [p.page_content for p in pages]
metadata = pd.DataFrame(index=list(range(len(texts))))
metadata["tag"] = "law"
metadata["title"] = "Déclaration des Droits de l'Homme et du Citoyen de 1789".encode("utf-8")
vectordb = KDBAI(table, embeddings)
vectordb.add_texts(texts=texts, metadatas=metadata)
```

```output
从 PDF 文本创建向量数据库...
CPU times: user 211 ms, sys: 18.4 ms, total: 229 ms
Wall time: 2.23 s
```

```output
['3ef27d23-47cf-419b-8fe9-5dfae9e8e895',
 'd3a9a69d-28f5-434b-b95b-135db46695c8',
 'd2069bda-c0b8-4791-b84d-0c6f84f4be34']
```

## 创建 LangChain Pipeline

```python
%%time
print("创建 LangChain Pipeline...")
qabot = RetrievalQA.from_chain_type(
    chain_type="stuff",
    llm=ChatOpenAI(model="gpt-3.5-turbo-16k", temperature=TEMP),
    retriever=vectordb.as_retriever(search_kwargs=dict(k=K)),
    return_source_documents=True,
)
```

```output
创建 LangChain Pipeline...
CPU times: user 40.8 ms, sys: 4.69 ms, total: 45.5 ms
Wall time: 44.7 ms
```

## 用英语总结文档

```python
%%time
Q = "用英语总结文档:"
print(f"\n\n{Q}\n")
print(qabot.invoke(dict(query=Q))["result"])
```

```output
用英语总结文档:
The document is the Declaration of the Rights of Man and of the Citizen of 1789. It was written by the representatives of the French people and aims to declare the natural, inalienable, and sacred rights of every individual. These rights include freedom, property, security, and resistance to oppression. The document emphasizes the importance of equality and the principle that sovereignty resides in the nation. It also highlights the role of law in protecting individual rights and ensuring the common good. The document asserts the right to freedom of thought, expression, and religion, as long as it does not disturb public order. It emphasizes the need for a public force to guarantee the rights of all citizens and the importance of a fair and equal distribution of public contributions. The document also recognizes the right of citizens to hold public officials accountable and states that any society without the guarantee of rights and separation of powers does not have a constitution. Finally, it affirms the inviolable and sacred nature of property, stating that it can only be taken away for public necessity and with just compensation.
CPU times: user 144 ms, sys: 50.2 ms, total: 194 ms
Wall time: 4.96 s
```

## 查询数据

```python
%%time
Q = "这是一项公平的法律吗？为什么？"
print(f"\n\n{Q}\n")
print(qabot.invoke(dict(query=Q))["result"])
```

```output
这是一项公平的法律吗？为什么？
作为一个人工智能语言模型，我没有个人观点。然而，我可以根据给定的背景提供一些分析。提供的文本摘自1789年《人权和公民权宣言》，被认为是人权历史上的基础文件。它概述了个人的自然和不可剥夺的权利，如自由、财产、安全和抵抗压迫。它还强调了平等原则、法治和权力分立。
这项法律是否被认为公平是主观的，可以根据个人观点和社会规范而变化。然而，许多人认为该宣言中概述的原则和权利是基本和公正的。重要的是要注意，这一宣言是法国确立平等原则和个人权利的重要一步，并影响了随后在全球范围内的人权文件。
CPU times: user 85.1 ms, sys: 5.93 ms, total: 91.1 ms
Wall time: 5.11 s
```

```python
%%time
Q = "人、公民和社会的权利和义务是什么？"
print(f"\n\n{Q}\n")
print(qabot.invoke(dict(query=Q))["result"])
```

```output
人、公民和社会的权利和义务是什么？
根据1789年《人权和公民权宣言》，人、公民和社会的权利和义务如下：
人的权利：
1. 人生而自由，且在权利上一律平等。社会的区分只能基于共同的效用。
2. 政治联合的目的是保护人的自然和不可剥夺的权利，即自由、财产、安全和抵抗压迫。
3. 主权原则本质上属于国家。任何团体或个人都不能行使不明确来源于它的权威。
4. 自由包括能够做任何不损害他人的事情。每个人对自然权利的行使没有其他限制，只有确保其他社会成员享有这些权利的限制，这些限制只能由法律确定。
5. 法律有权禁止只有对社会有害的行为。法律未禁止的事情不能被阻止，也不能强迫任何人去做它不命令的事情。
6. 法律是一种普遍意志的表达。所有公民都有权亲自参与或通过他们的代表参与其形成。无论是保护还是惩罚，对于所有公民来说都应该是一样的。所有公民在它的眼中是平等的，都有资格担任一切公共职务、地位和职业，根据他们的能力，而没有其他区别，只有他们的美德和才能。
7. 除非法律规定的情况和法律规定的形式，否则任何人都不能被指控、逮捕或拘留。那些请求、加速、执行或导致执行任意命令的人必须受到惩罚。但是，任何根据法律被召唤或被拘留的公民必须立即服从；他通过抵抗使自己有罪。
8. 法律应该只规定严格和明显必要的惩罚，除非根据在犯罪之前制定和公布的法律，否则任何人都不能受到惩罚。
9. 每个人被推定为无罪，直到被宣判有罪，如果认为必须逮捕他，任何不必要的严厉对待必须受到法律的严厉打击。
10. 任何人的意见，甚至宗教意见，只要它们的表现不通过法律扰乱既定的公共秩序，就不应该受到干扰。
11. 思想和意见的自由交流是人的最宝贵的权利之一。因此，每个公民都可以自由地说话、写作和印刷，除了在法律规定的情况下对这种自由的滥用作出回应。
12. 保障人和公民的权利需要有一种公共力量。因此，这种力量是为了所有人的利益而设立的，而不是为了委托给它的人的特殊利益。
13. 为了维持公共力量和行政开支，需要共同的贡献。它必须在所有公民中平均分配，按照他们的能力。
14. 所有公民都有权亲自或通过他们的代表确定公共贡献的必要性，自由同意，跟踪其使用，并确定其金额、基础、征收和期限。
15. 社会有权要求任何公共代理人对他的管理负责。
```

**16. 没有确保权利保障，也没有确定权力分立的社会，就没有宪法。**

**17. 财产是不可侵犯和神圣的权利，除非经法律确定的公共需要明显需要，并在公正和事先补偿的条件下，否则任何人都不能被剥夺。**

公民的责任：

《宣言》并未明确提及公民的责任，但强调了法律的重要性，以及公民遵守法律的必要性。它还强调了公民参与法律形成的责任，并要求公民对公共代理人的行政行为负责。

社会的责任：

《宣言》并未明确提及社会的责任，但强调了保障人和公民的权利的重要性。社会有责任确保这些权利和权力的分立，并提供公共力量，使其造福于所有人。

**Q: 这项法律是否切实可行？**

作为一个AI语言模型，我无法就一项法律是否切实可行提供个人意见或主观判断。所提供的文本摘自1789年的《人权和公民权宣言》和法国宪法。这些文本概述了构成法国法律体系基础的基本权利和原则。一项法律的切实可行性往往是一种解释问题，可以根据具体情况和特定环境而变化。最终，法律专家、立法者和司法机构将决定这些法律在具体案例中的切实可行性和适用性。