# Vearch

[Vearch](https://vearch.readthedocs.io) 是用于深度学习和人工智能应用的向量搜索基础设施。

## 设置

请按照[说明](https://vearch.readthedocs.io/en/latest/quick-start-guide.html#)进行设置。

```python
%pip install --upgrade --quiet  vearch
# 或者
%pip install --upgrade --quiet  vearch_cluster
```

## 示例

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.vearch import Vearch
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
from transformers import AutoModel, AutoTokenizer
# 替换为您的本地模型路径
model_path = "/data/zhx/zhx/langchain-ChatGLM_new/chatglm2-6b"
tokenizer = AutoTokenizer.from_pretrained(model_path, trust_remote_code=True)
model = AutoModel.from_pretrained(model_path, trust_remote_code=True).half().cuda(0)
```
```output
加载检查点分片：100%|██████████| 7/7 [00:07<00:00,  1.01s/it]
```
```python
query = "你好!"
response, history = model.chat(tokenizer, query, history=[])
print(f"人类: {query}\nChatGLM:{response}\n")
query = "你知道凌波微步吗，你知道都有谁学会了吗?"
response, history = model.chat(tokenizer, query, history=history)
print(f"人类: {query}\nChatGLM:{response}\n")
```
```output
人类: 你好!
ChatGLM:你好👋！我是人工智能助手 ChatGLM2-6B，很高兴见到你，欢迎问我任何问题。
人类: 你知道凌波微步吗，你知道都有谁学会了吗?
ChatGLM:凌波微步是一种步伐，最早出自《倚天屠龙记》。在电视剧《人民的名义》中，侯亮平也学会了凌波微步。
```
```python
# 添加您的本地知识文件
file_path = "/data/zhx/zhx/langchain-ChatGLM_new/knowledge_base/天龙八部/lingboweibu.txt"  # 您的本地文件路径"
loader = TextLoader(file_path, encoding="utf-8")
documents = loader.load()
# 将文本拆分为句子并嵌入句子
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=100)
texts = text_splitter.split_documents(documents)
# 替换为您的模型路径
embedding_path = "/data/zhx/zhx/langchain-ChatGLM_new/text2vec/text2vec-large-chinese"
embeddings = HuggingFaceEmbeddings(model_name=embedding_path)
```
```output
未找到带有名称 /data/zhx/zhx/langchain-ChatGLM_new/text2vec/text2vec-large-chinese 的句子转换模型。正在使用 MEAN 池化创建一个新模型。
```
```python
# 首先将您的文档添加到 vearch 向量存储
vearch_standalone = Vearch.from_documents(
    texts,
    embeddings,
    path_or_url="/data/zhx/zhx/langchain-ChatGLM_new/knowledge_base/localdb_new_test",
    table_name="localdb_new_test",
    flag=0,
)
print("***************以下是集群结果*****************")
vearch_cluster = Vearch.from_documents(
    texts,
    embeddings,
    path_or_url="http://test-vearch-langchain-router.vectorbase.svc.ht1.n.jd.local",
    db_name="vearch_cluster_langchian",
    table_name="tobenumone",
    flag=1,
)
```
```output
文档 ID ['18ce6747dca04a2c833e60e8dfd83c04', 'aafacb0e46574b378a9f433877ab06a8', '9776bccfdd8643a8b219ccee0596f370']
***************以下是集群结果*****************
文档 ID ['1841638988191686991', '-4519586577642625749', '5028230008472292907']
```
```python
query = "你知道凌波微步吗，你知道都有谁会凌波微步?"
vearch_standalone_res = vearch_standalone.similarity_search(query, 3)
for idx, tmp in enumerate(vearch_standalone_res):
    print(f"{'#'*20}第{idx+1}段相关文档{'#'*20}\n\n{tmp.page_content}\n")
# 结合您的本地知识和查询
context = "".join([tmp.page_content for tmp in vearch_standalone_res])
new_query = f"基于以下信息，尽可能准确地回答用户的问题。背景信息:\n {context} \n 回答用户这个问题:{query}\n\n"
response, history = model.chat(tokenizer, new_query, history=[])
print(f"********ChatGLM:{response}\n")
print("***************************以下是集群结果******************************")
query_c = "你知道凌波微步吗，你知道都有谁会凌波微步?"
cluster_res = vearch_cluster.similarity_search(query_c, 3)
for idx, tmp in enumerate(cluster_res):
    print(f"{'#'*20}第{idx+1}段相关文档{'#'*20}\n\n{tmp.page_content}\n")
# 结合您的本地知识和查询
context_c = "".join([tmp.page_content for tmp in cluster_res])
new_query_c = f"基于以下信息，尽可能准确地回答用户的问题。背景信息:\n {context_c} \n 回答用户这个问题:{query_c}\n\n"
response_c, history_c = model.chat(tokenizer, new_query_c, history=[])
print(f"********ChatGLM:{response_c}\n")
```
```output
####################第1段相关文档####################
午饭过后，段誉又练“凌波微步”，走一步，吸一口气，走第二步时将气呼出，六十四卦走完，四肢全无麻痹之感，料想呼吸顺畅，便无害处。第二次再走时连走两步吸一口气，再走两步始行呼出。这“凌波微步”是以动功修习内功，脚步踏遍六十四卦一个周天，内息自然而然地也转了一个周天。因此他每走一遍，内力便有一分进益。
这般练了几天，“凌波微步”已走得颇为纯熟，不须再数呼吸，纵然疾行，气息也已无所窒滞。心意既畅，跨步时渐渐想到《洛神赋》中那些与“凌波微步”有关的句子：“仿佛兮若轻云之蔽月，飘飘兮若流风之回雪”，“竦轻躯以鹤立，若将飞而未翔”，“体迅飞凫，飘忽若神”，“动无常则，若危若安。进止难期，若往若还”。
百度简介
凌波微步是「逍遥派」独门轻功身法，精妙异常。
凌波微步乃是一门极上乘的轻功，所以列于卷轴之末，以易经八八六十四卦为基础，使用者按特定顺序踏着卦象方位行进，从第一步到最后一步正好行走一个大圈。此步法精妙异常，原是要待人练成「北冥神功」，吸人内力，自身内力已【颇为深厚】之后再练。
####################第2段相关文档####################
《天龙八部》第五回 微步縠纹生
卷轴中此外诸种经脉修习之法甚多，皆是取人内力的法门，段誉虽自语宽解，总觉习之有违本性，单是贪多务得，便非好事，当下暂不理会。
卷到卷轴末端，又见到了“凌波微步”那四字，登时便想起《洛神赋》中那些句子来：“凌波微步，罗袜生尘……转眄流精，光润玉颜。含辞未吐，气若幽兰。华容婀娜，令我忘餐。”曹子建那些千古名句，在脑海中缓缓流过：“秾纤得衷，修短合度，肩若削成，腰如约素。延颈秀项，皓质呈露。芳泽无加，铅华弗御。云髻峨峨，修眉连娟。丹唇外朗，皓齿内鲜。明眸善睐，靥辅承权。瑰姿艳逸，仪静体闲。柔情绰态，媚于语言……”这些句子用在木婉清身上，“这话倒也有理”；但如用之于神仙姊姊，只怕更为适合。想到神仙姊姊的姿容体态，“皎若太阳升朝霞，灼若芙蓉出绿波”，但觉依她吩咐行事，实为人生至乐，心想：“我先来练这‘凌波微步’，此乃逃命之妙法，非害人之手段也，练之有百利而无一害。”
####################第3段相关文档####################
《天龙八部》第二回 玉壁月华明
再展帛卷，长卷上源源皆是裸女画像，或立或卧，或现前胸，或见后背。人像的面容都是一般，但或喜或愁，或含情凝眸，或轻嗔薄怒，神情各异。一共有三十六幅图像，每幅像上均有颜色细线，注明穴道部位及练功法诀。
帛卷尽处题着“凌波微步”四字，其后绘的是无数足印，注明“妇妹”、“无妄”等等字样，尽是《易经》中的方位。段誉前几日还正全心全意地钻研《易经》，一见到这些名称，登时精神大振，便似遇到故交良友一般。只见足印密密麻麻，不知有几千百个，自一个足印至另一个足印均有绿线贯串，线上绘有箭头，最后写着一行字道：“步法神妙，保身避敌，待积内力，再取敌命。”
段誉心道：“神仙姊姊所遗的步法，必定精妙之极，遇到强敌时脱身逃走，那就很好，‘再取敌命’也就不必了。”
卷好帛卷，对之作了两个揖，珍而重之地揣入怀中，转身对那玉像道：“神仙姊姊，你吩咐我朝午晚三次练功，段誉不敢有违。今后我对人加倍客气，别人不会来打我，我自然也不会去吸他内力。你这套‘凌波微步’我更要用心练熟，眼见不对，立刻溜之大吉，就吸不到他内力了。”至于“杀尽我逍遥派弟子”一节，却想也不敢去想。
********ChatGLM:凌波微步是一门极上乘的轻功，源于《易经》八八六十四卦。使用者按照特定顺序踏着卦象方位行进，从第一步到最后一步正好行走一个大圈。这门轻功精妙异常，可以使人内力大为提升，但需在练成“北冥神功”后才能真正掌握。凌波微步在金庸先生的《天龙八部》中得到了充分的描写。
***************************以下是集群结果******************************
####################第1段相关文档####################
午饭过后，段誉又练“凌波微步”，走一步，吸一口气，走第二步时将气呼出，六十四卦走完，四肢全无麻痹之感，料想呼吸顺畅，便无害处。第二次再走时连走两步吸一口气，再走两步始行呼出。这“凌波微步”是以动功修习内功，脚步踏遍六十四卦一个周天，内息自然而然地也转了一个周天。因此他每走一遍，内力便有一分进益。
这般练了几天，“凌波微步”已走得颇为纯熟，不须再数呼吸，纵然疾行，气息也已无所窒滞。心意既畅，跨步时渐渐想到《洛神赋》中那些与“凌波微步”有关的句子：“仿佛兮若轻云之蔽月，飘飘兮若流风之回雪”，“竦轻躯以鹤立，若将飞而未翔”，“体迅飞凫，飘忽若神”，“动无常则，若危若安。进止难期，若往若还”。
百度简介
凌波微步是「逍遥派」独门轻功身法，精妙异常。
凌波微步乃是一门极上乘的轻功，所以列于卷轴之末，以易经八八六十四卦为基础，使用者按特定顺序踏着卦象方位行进，从第一步到最后一步正好行走一个大圈。此步法精妙异常，原是要待人练成「北冥神功」，吸人内力，自身内力已【颇为深厚】之后再练。
####################第2段相关文档####################
《天龙八部》第五回 微步縠
```python
query = "你知道vearch是什么吗?"
response, history = model.chat(tokenizer, query, history=history)
print(f"Human: {query}\nChatGLM:{response}\n")
vearch_info = [
    "Vearch 是一款存储大语言模型数据的向量数据库，用于存储和快速搜索模型embedding后的向量，可用于基于个人知识库的大模型应用",
    "Vearch 支持OpenAI, Llama, ChatGLM等模型，以及LangChain库",
    "vearch 是基于C语言,go语言开发的，并提供python接口，可以直接通过pip安装",
]
vearch_source = [
    {
        "source": "/data/zhx/zhx/langchain-ChatGLM_new/knowledge_base/tlbb/three_body.txt"
    },
    {
        "source": "/data/zhx/zhx/langchain-ChatGLM_new/knowledge_base/tlbb/three_body.txt"
    },
    {
        "source": "/data/zhx/zhx/langchain-ChatGLM_new/knowledge_base/tlbb/three_body.txt"
    },
]
vearch_standalone.add_texts(vearch_info, vearch_source)
print("*****************after is cluster res********************")
vearch_cluster.add_texts(vearch_info, vearch_source)
```
```output

Human: 你知道vearch是什么吗?

ChatGLM:是的，Vearch是一种用于存储大型语言模型数据的向量数据库，用于存储和快速搜索模型嵌入后的向量，可用于基于个人知识库的大型模型应用。

docids ['eee5e7468434427eb49829374c1e8220', '2776754da8fc4bb58d3e482006010716', '9223acd6d89d4c2c84ff42677ac0d47c']

*****************after is cluster res********************

docids ['-4311783201092343475', '-2899734009733762895', '1342026762029067927']

```
```output

['-4311783201092343475', '-2899734009733762895', '1342026762029067927']

```
```python
query3 = "你知道vearch是什么吗?"
res1 = vearch_standalone.similarity_search(query3, 3)
for idx, tmp in enumerate(res1):
    print(f"{'#'*20}第{idx+1}段相关文档{'#'*20}\n\n{tmp.page_content}\n")
context1 = "".join([tmp.page_content for tmp in res1])
new_query1 = f"基于以下信息，尽可能准确的来回答用户的问题。背景信息:\n {context1} \n 回答用户这个问题:{query3}\n\n"
response, history = model.chat(tokenizer, new_query1, history=[])
print(f"***************ChatGLM:{response}\n")
print("***************after is cluster res******************")
query3_c = "你知道vearch是什么吗?"
res1_c = vearch_standalone.similarity_search(query3_c, 3)
for idx, tmp in enumerate(res1_c):
    print(f"{'#'*20}第{idx+1}段相关文档{'#'*20}\n\n{tmp.page_content}\n")
context1_C = "".join([tmp.page_content for tmp in res1_c])
new_query1_c = f"基于以下信息，尽可能准确的来回答用户的问题。背景信息:\n {context1_C} \n 回答用户这个问题:{query3_c}\n\n"
response_c, history_c = model.chat(tokenizer, new_query1_c, history=[])
print(f"***************ChatGLM:{response_c}\n")
```
```output

####################第1段相关文档####################

Vearch 是一款存储大语言模型数据的向量数据库，用于存储和快速搜索模型embedding后的向量，可用于基于个人知识库的大模型应用

####################第2段相关文档####################

Vearch 支持OpenAI, Llama, ChatGLM等模型，以及LangChain库

####################第3段相关文档####################

vearch 是基于C语言,go语言开发的，并提供python接口，可以直接通过pip安装

***************ChatGLM:是的，Vearch是一种用于存储大型语言模型数据的向量数据库，用于存储和快速搜索模型嵌入后的向量，可用于基于个人知识库的大型模型应用。它支持OpenAI、Llama、ChatGLM等模型，并提供Python接口，可以直接通过pip安装。

***************after is cluster res******************

####################第1段相关文档####################

Vearch 是一款存储大语言模型数据的向量数据库，用于存储和快速搜索模型embedding后的向量，可用于基于个人知识库的大模型应用

####################第2段相关文档####################

Vearch 支持OpenAI, Llama, ChatGLM等模型，以及LangChain库

####################第3段相关文档####################

vearch 是基于C语言,go语言开发的，并提供python接口，可以直接通过pip安装

***************ChatGLM:是的，Vearch是一种用于存储大型语言模型数据的向量数据库，用于存储和快速搜索模型嵌入后的向量，可用于基于个人知识库的大型模型应用。它支持OpenAI、Llama、ChatGLM等模型，并提供Python接口，可以直接通过pip安装。

```
```python
##delete and get function need to maintian  docids
##your docid
res_d = vearch_standalone.delete(
    [
        "eee5e7468434427eb49829374c1e8220",
        "2776754da8fc4bb58d3e482006010716",
        "9223acd6d89d4c2c84ff42677ac0d47c",
    ]
)
print("delete vearch standalone docid", res_d)
query = "你知道vearch是什么吗?"
response, history = model.chat(tokenizer, query, history=[])
print(f"Human: {query}\nChatGLM:{response}\n")
res_cluster = vearch_cluster.delete(
    ["-4311783201092343475", "-2899734009733762895", "1342026762029067927"]
)
print("delete vearch cluster docid", res_cluster)
query_c = "你知道vearch是什么吗?"
response_c, history = model.chat(tokenizer, query_c, history=[])
print(f"Human: {query}\nChatGLM:{response_c}\n")
get_delet_doc = vearch_standalone.get(
    [
        "eee5e7468434427eb49829374c1e8220",
        "2776754da8fc4bb58d3e482006010716",
        "9223acd6d89d4c2c84ff42677ac0d47c",
    ]
)
print("after delete docid to query again:", get_delet_doc)
get_id_doc = vearch_standalone.get(
    [
        "18ce6747dca04a2c833e60e8dfd83c04",
        "aafacb0e46574b378a9f433877ab06a8",
        "9776bccfdd8643a8b219ccee0596f370",
        "9223acd6d89d4c2c84ff42677ac0d47c",
    ]
)
print("get existed docid", get_id_doc)
get_delet_doc = vearch_cluster.get(
    ["-4311783201092343475", "-2899734009733762895", "1342026762029067927"]
)
print("after delete docid to query again:", get_delet_doc)
get_id_doc = vearch_cluster.get(
    [
        "1841638988191686991",
        "-4519586577642625749",
        "5028230008472292907",
```
```python
"1342026762029067927",
]
)
print("获取已存在的文档id", get_id_doc)
```
```output

删除 vearch 独立文档id True

Human: 你知道vearch是什么吗?

ChatGLM: Vearch是一种用于处理向量的库，可以轻松地将向量转换为矩阵，并提供许多有用的函数和算法，以操作向量。 Vearch支持许多常见的向量操作，例如加法、减法、乘法、除法、矩阵乘法、求和、统计和归一化等。 Vearch还提供了一些高级功能，例如L2正则化、协方差矩阵、稀疏矩阵和奇异值分解等。

删除 vearch 集群文档id True

Human: 你知道vearch是什么吗?

ChatGLM: Vearch是一种用于处理向量数据的库，可以应用于多种不同的编程语言和数据结构中。

Vearch最初是作为Java中一个名为“vearch”的包而出现的，它的目的是提供一种高效的向量数据结构。它支持向量的多态性，可以轻松地实现不同类型的向量之间的转换，同时还支持向量的压缩和反向操作等操作。

后来，Vearch被广泛应用于其他编程语言中，如Python、Ruby、JavaScript等。在Python中，它被称为“vectorize”，在Ruby中，它被称为“Vector”。

Vearch的主要优点是它的向量操作具有多态性，可以应用于不同类型的向量数据，同时还支持高效的向量操作和反向操作，因此可以提高程序的性能。

删除文档id后再次查询：{}

获取已存在的文档id {'18ce6747dca04a2c833e60e8dfd83c04': Document(page_content='《天龙八部》第二回 玉壁月华明\n\n再展帛卷，长卷上源源皆是裸女画像，或立或卧，或现前胸，或见后背。人像的面容都是一般，但或喜或愁，或含情凝眸，或轻嗔薄怒，神情各异。一共有三十六幅图像，每幅像上均有颜色细线，注明穴道部位及练功法诀。\n\n帛卷尽处题着“凌波微步”四字，其后绘的是无数足印，注明“妇妹”、“无妄”等等字样，尽是《易经》中的方位。段誉前几日还正全心全意地钻研《易经》，一见到这些名称，登时精神大振，便似遇到故交良友一般。只见足印密密麻麻，不知有几千百个，自一个足印至另一个足印均有绿线贯串，线上绘有箭头，最后写着一行字道：“步法神妙，保身避敌，待积内力，再取敌命。”\n\n段誉心道：“神仙姊姊所遗的步法，必定精妙之极，遇到强敌时脱身逃走，那就很好，‘再取敌命’也就不必了。”\n卷好帛卷，对之作了两个揖，珍而重之地揣入怀中，转身对那玉像道：“神仙姊姊，你吩咐我朝午晚三次练功，段誉不敢有违。今后我对人加倍客气，别人不会来打我，我自然也不会去吸他内力。你这套‘凌波微步’我更要用心练熟，眼见不对，立刻溜之大吉，就吸不到他内力了。”至于“杀尽我逍遥派弟子”一节，却想也不敢去想。', metadata={'source': '/data/zhx/zhx/langchain-ChatGLM_new/knowledge_base/天龙八部/lingboweibu.txt'}), 'aafacb0e46574b378a9f433877ab06a8': Document(page_content='《天龙八部》第五回 微步縠纹生\n\n卷轴中此外诸种经脉修习之法甚多，皆是取人内力的法门，段誉虽自语宽解，总觉习之有违本性，单是贪多务得，便非好事，当下暂不理会。\n\n卷到卷轴末端，又见到了“凌波微步”那四字，登时便想起《洛神赋》中那些句子来：“凌波微步，罗袜生尘……转眄流精，光润玉颜。含辞未吐，气若幽兰。华容婀娜，令我忘餐。”曹子建那些千古名句，在脑海中缓缓流过：“秾纤得衷，修短合度，肩若削成，腰如约素。延颈秀项，皓质呈露。芳泽无加，铅华弗御。云髻峨峨，修眉连娟。丹唇外朗，皓齿内鲜。明眸善睐，靥辅承权。瑰姿艳逸，仪静体闲。柔情绰态，媚于语言……”这些句子用在木婉清身上，“这话倒也有理”；但如用之于神仙姊姊，只怕更为适合。想到神仙姊姊的姿容体态，“皎若太阳升朝霞，灼若芙蓉出绿波”，但觉依她吩咐行事，实为人生至乐，心想：“我先来练这‘凌波微步’，此乃逃命之妙法，非害人之手段也，练之有百利而无一害。”', metadata={'source': '/data/zhx/zhx/langchain-ChatGLM_new/knowledge_base/天龙八部/lingboweibu.txt'}), '9776bccfdd8643a8b219ccee0596f370': Document(page_content='午饭过后，段誉又练“凌波微步”，走一步，吸一口气，走第二步时将气呼出，六十四卦走完，四肢全无麻痹之感，料想呼吸顺畅，便无害处。第二次再走时连走两步吸一口气，再走两步始行呼出。这“凌波微步”是以动功修习内功，脚步踏遍六十四卦一个周天，内息自然而然地也转了一个周天。因此他每走一遍，内力便有一分进益。\n\n这般练了几天，“凌波微步”已走得颇为纯熟，不须再数呼吸，纵然疾行，气息也已无所窒滞。心意既畅，跨步时渐渐想到《洛神赋》中那些与“凌波微步”有关的句子：“仿佛兮若轻云之蔽月，飘飘兮若流风之回雪”，“竦轻躯以鹤立，若将飞而未翔”，“体迅飞凫，飘忽若神”，“动无常则，若危若安。进止难期，若往若还”。\n\n\n\n百度简介\n\n凌波微步是「逍遥派」独门轻功身法，精妙异常。\n\n凌波微步乃是一门极上乘的轻功，所以列于卷轴之末，以易经八八六十四卦为基础，使用者按特定顺序踏着卦象方位行进，从第一步到最后一步正好行走一个大圈。此步法精妙异常，原是要待人练成「北冥神功」，吸人内力，自身内力已【颇为深厚】之后再练。', metadata={'source': '/data/zhx/zhx/langchain-ChatGLM_new/knowledge_base/天龙八部/lingboweibu.txt'})}

删除文档id后再次查询：{}

获取已存在的文档id {'1841638988191686991': Document(page_content='《天龙八部》第二回 玉壁月华明\n\n再展帛卷，长卷上源源皆是裸女画像，或立或卧，或现前胸，或见后背。人像的面容都是一般，但或喜或愁，或含情凝眸，或轻嗔薄怒，神情各异。一共有三十六幅图像，每幅像上均有颜色细线，注明穴道部位及练功法诀。\n\n帛卷尽处题着“凌波微步”四字，其后绘的是无数足印，注明“妇妹”、“无妄”等等字样，尽是《易经》中的方位。段誉前几日还正全心全意地钻研《易经》，一见到这些名称，登时精神大振，便似遇到故交良友一般。只见足印密密麻麻，不知有几千百个，自一个足印至另一个足印均有绿线贯串，线上绘有箭头，最后写着一行字道：“步法神妙，保身避敌，待积内力，再取敌命。”\n\n段誉心道：“神仙姊姊所遗的步法，必定精妙之极，遇到强敌时脱身逃走，那就很好，‘再取敌命’也就不必了。”\n卷好帛卷，对之作了两个揖，珍而重之地揣入怀中，转身对那玉像道：“神仙姊姊，你吩咐我朝午晚三次练功，段誉不敢有违。今后我对人加倍客气，别人不会来打我，我自然也不会去吸他内力。你这套‘凌波微步’我更要用心练熟，眼见不对，立刻溜之大吉，就吸不到他内力了。”至于“杀尽我逍遥派弟子”一节，却想也不敢去想。', metadata={'source': '/data/zhx/zhx/langchain-ChatGLM_new/knowledge_base/天龙八部/lingboweibu.txt'}), '-4519586577642625749': Document(page_content='《天龙八部》第五回 微步縠纹生\n\n卷轴中此外诸种经脉修习之法甚多，皆是取人内力的法门，段誉虽自语宽解，总觉习之有违本性，单是贪多务得，便非好事，当下暂不理会。\n\n卷到卷轴末端，又见到了“凌波微步”那四字，登时便想起《洛神赋》中那些句子来：“凌波微步，罗袜生尘……转眄流精，光润玉颜。含辞未吐，气若幽兰。华容婀娜，令我忘餐。”曹子建那些千古名句，在脑海中缓缓流过：“秾纤得衷，修短合度，肩若削成，腰如约素。延颈秀项，皓质呈露。芳泽无加，铅华弗御。云髻峨峨，修眉连娟。丹唇外朗，皓齿内鲜。明眸善睐，靥辅承权。瑰姿艳逸，仪静体闲。柔情绰态，媚于语言……”这些句子用在木婉清身上，“这话倒也有理”；但如用之于神仙姊姊，只怕更为适合。想到神仙姊姊的姿容体态，“皎若太阳升朝霞，灼若芙蓉出绿波”，但觉依她吩咐行事，实为人生至乐，心想：“我先来练这‘凌波微步’，此乃逃命之妙法，非害人之手段也，练之有百利而无一害。”', metadata={'source': '/data/zhx/zhx/langchain-ChatGLM_new/knowledge_base/天龙八部/lingboweibu.txt'}), '5028230008472292907': Document(page_content='午饭过后，段誉又练“凌波微步”，走一步，吸一口气，走第二步时将气呼出，六十四卦走完，四肢全无麻痹之感，料想呼吸顺畅，便无害处。第二次再走时连走两步吸一口气，再走两步始行呼出。这“凌波微步”是以动功修习内功，脚步踏遍六十四卦一个周天，内息自然而然地也转了一个周天。因此他每走一遍，内力便有一分进益。\n\n这般练了几天，“凌波微步”已走得颇为纯熟，不须再数呼吸，纵然疾行，气息也已无所窒滞。心意既畅，跨步时渐渐想