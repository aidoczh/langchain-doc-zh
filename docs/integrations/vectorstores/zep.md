# Zep

> 回忆、理解并从聊天记录中提取数据。为个性化的 AI 体验提供动力。

> [Zep](https://www.getzep.com) 是一项长期记忆服务，适用于 AI 助手应用程序。

> 有了 Zep，您可以让 AI 助手具备回忆过去对话的能力，无论多久以前，

> 同时还能减少幻觉、延迟和成本。

> 感兴趣 Zep Cloud 吗？请参阅 [Zep Cloud 安装指南](https://help.getzep.com/sdks) 和 [Zep Cloud 向量存储示例](https://help.getzep.com/langchain/examples/vectorstore-example)

## 开源安装和设置

> Zep 开源项目：[https://github.com/getzep/zep](https://github.com/getzep/zep)

>

> Zep 开源文档：[https://docs.getzep.com/](https://docs.getzep.com/)

## 用法

在下面的示例中，我们使用了 Zep 的自动嵌入功能，该功能会自动在 Zep 服务器上嵌入文档，使用低延迟嵌入模型。

## 注意

- 这些示例使用了 Zep 的异步接口。通过从方法名称中删除 `a` 前缀，可以调用同步接口。

- 如果您传入一个 `Embeddings` 实例，Zep 将使用它来嵌入文档，而不是自动嵌入它们。

您还必须将文档集合设置为 `isAutoEmbedded === false`。

- 如果您将集合设置为 `isAutoEmbedded === false`，您必须传入一个 `Embeddings` 实例。

## 从文档加载或创建集合

```python
from uuid import uuid4
from langchain_community.document_loaders import WebBaseLoader
from langchain_community.vectorstores import ZepVectorStore
from langchain_community.vectorstores.zep import CollectionConfig
from langchain_text_splitters import RecursiveCharacterTextSplitter
ZEP_API_URL = "http://localhost:8000"  # 这是您的 Zep 实例的 API URL
ZEP_API_KEY = "<optional_key>"  # 您的 Zep 实例的可选 API 密钥
collection_name = f"babbage{uuid4().hex}"  # 唯一的集合名称。仅限字母数字
# 如果我们要创建一个新的 Zep 集合，则需要集合配置
config = CollectionConfig(
    name=collection_name,
    description="<optional description>",
    metadata={"optional_metadata": "associated with the collection"},
    is_auto_embedded=True,  # 我们将让 Zep 使用其低延迟嵌入器嵌入我们的文档
    embedding_dimensions=1536,  # 这应该与您配置 Zep 使用的模型相匹配。
)
# 加载文档
article_url = "https://www.gutenberg.org/cache/epub/71292/pg71292.txt"
loader = WebBaseLoader(article_url)
documents = loader.load()
# 将其分成块
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
# 实例化 VectorStore。由于集合在 Zep 中尚不存在，
# 它将被创建并用我们传入的文档填充。
vs = ZepVectorStore.from_documents(
    docs,
    collection_name=collection_name,
    config=config,
    api_url=ZEP_API_URL,
    api_key=ZEP_API_KEY,
    embedding=None,  # 我们将让 Zep 使用其低延迟嵌入器嵌入我们的文档
)
```
```python
# 等待集合嵌入完成
async def wait_for_ready(collection_name: str) -> None:
    import time
    from zep_python import ZepClient
    client = ZepClient(ZEP_API_URL, ZEP_API_KEY)
    while True:
        c = await client.document.aget_collection(collection_name)
        print(
            "嵌入状态: "
            f"{c.document_embedded_count}/{c.document_count} 已嵌入文档"
        )
        time.sleep(1)
        if c.status == "ready":
            break
await wait_for_ready(collection_name)
```
```output
嵌入状态: 0/401 已嵌入文档
嵌入状态: 0/401 已嵌入文档
嵌入状态: 0/401 已嵌入文档
嵌入状态: 0/401 已嵌入文档
嵌入状态: 0/401 已嵌入文档
嵌入状态: 0/401 已嵌入文档
嵌入状态: 401/401 已嵌入文档
```

## 对集合进行相似性搜索查询

```python
# 查询
query = "太阳系的结构是什么？"
docs_scores = await vs.asimilarity_search_with_relevance_scores(query, k=3)
# 打印结果
for d, s in docs_scores:
    print(d.page_content, " -> ", s, "\n====\n")
```
```output
两个主要行星的位置（这对于航海者来说是最重要的）——木星和土星，每个都需要不少于一百一十六张表。然而，不仅需要预测这些天体的位置，而且还需要制表预测木星的四颗卫星的运动，预测它们进入阴影的确切时间，以及它们的阴影穿过其盘的时间，以及它们相互交叉的时间  ->  0.9003241539387915 
====
提供的表格不仅不足以为导航（在广义上的意义上）提供更多帮助，而且在计算和打印表格方面更加便利、迅速和经济的情况下，它可能提供更多的帮助。
用于确定行星位置的表格不仅不比太阳、月亮和星星的表格少，而是更为必要。当我们陈述这些表格的数量和复杂性时，就可以形成对它们的一些概念，因为  ->  0.8911165633479508 
====
因此，立即应用这种方案的记号方案，立即提出了必须遵守的优点，作为表达动物系统的结构、运作和循环的工具；我们对它的适用性毫无疑问。不仅是人和动物的固体成员的机械连接，而且还有软件部分的结构和运作，包括肌肉、外皮、膜等，其性质、运动  ->  0.8899750214770481 
====
## 通过MMR重新排序的集合搜索
Zep提供了本地的、硬件加速的MMR搜索结果重新排序。
```python
query = "太阳系的结构是什么？"
docs = await vs.asearch(query, search_type="mmr", k=3)
for d in docs:
    print(d.page_content, "\n====\n")
```
```output

两个主要行星的位置（对于航海者来说最重要的行星）——木星和土星，每个行星都需要不少于116张表格。然而，不仅需要预测这些天体的位置，还需要制表记录木星的四颗卫星的运动，以预测它们进入木星阴影的确切时间，以及它们的阴影穿过木星盘的时间，以及它们的干涉时间。

====

应用这种符号方案，立即就能想到它作为表达动物系统的结构、运作和循环的工具所带来的优势；我们对它在这个目的上的适用性毫不怀疑。不仅是人体和动物的固体成员的机械连接，而且包括肌肉、皮肤、膜等软组织的结构和运作，以及它们的性质、运动等。

====

阻力、节约时间、协调机制，并使整个机械动作达到最大的实际完美。

在这个机械装置中，为了实现所描述的结果，采用了一系列机械构思；尽管它们的动作非常完美，以至于在任何普通情况下，人们都会认为它们已经以几乎多余的程度达到了预期的目标。

====

```
# 通过元数据进行过滤
使用元数据过滤器来缩小结果范围。首先，加载另一本书："福尔摩斯探案集"
```python
# 让我们向现有集合添加更多内容
article_url = "https://www.gutenberg.org/files/48320/48320-0.txt"
loader = WebBaseLoader(article_url)
documents = loader.load()
# 将其分成块
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
await vs.aadd_documents(docs)
await wait_for_ready(collection_name)
```
```output

嵌入状态：已嵌入401/1691个文档

嵌入状态：已嵌入401/1691个文档

嵌入状态：已嵌入401/1691个文档

嵌入状态：已嵌入401/1691个文档

嵌入状态：已嵌入401/1691个文档

嵌入状态：已嵌入401/1691个文档

嵌入状态：已嵌入901/1691个文档

嵌入状态：已嵌入901/1691个文档

嵌入状态：已嵌入901/1691个文档

嵌入状态：已嵌入901/1691个文档

嵌入状态：已嵌入901/1691个文档

嵌入状态：已嵌入901/1691个文档

嵌入状态：已嵌入1401/1691个文档

嵌入状态：已嵌入1401/1691个文档

嵌入状态：已嵌入1401/1691个文档

嵌入状态：已嵌入1401/1691个文档

嵌入状态：已嵌入1691/1691个文档

```
我们看到了来自两本书的结果。注意`source`元数据。
```python
query = "他对天文学感兴趣吗？"
docs = await vs.asearch(query, search_type="similarity", k=3)
for d in docs:
    print(d.page_content, " -> ", d.metadata, "\n====\n")
```
```output

为此目的，可以使用这些表格进行远程或直接的测量。但除此之外，还需要大量专门用于天文学的表格。天文学家对于天体的位置和运动的预测是使航海者能够从事航海艺术的手段，也是唯一的手段。通过这些预测，他能够发现船只距离赤道的距离和范围。

 ->  {'source': 'https://www.gutenberg.org/cache/epub/71292/pg71292.txt'} 

====

拥有对他的工作有用的所有知识，这是我努力做到的。如果我记得没错，你在我们友谊的早期某个场合上，以非常明确的方式定义了我的限制。”

“是的，”我笑着回答道。“那是一份奇特的文件。哲学、天文学和政治被标记为零，我记得。植物学可变，地质学则对任何地区的泥土污渍非常深刻。

 ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 

====

天文学及其相关科学以及依赖于它们的各种艺术。在没有计算机的情况下，没有比天文学更费力的计算了；在没有准备设施的情况下，没有比天文学更需要准备设施的了；在没有错误更有害的情况下，没有比天文学更需要错误更有害的情况的了。实际的天文学家在进行观测任务时，会被繁琐的计算工作所打断和分散注意力。

====

```
现在，我们设置了一个过滤器
```python
filter = {
    "where": {
        "jsonpath": (
            "$[*] ? (@.source == 'https://www.gutenberg.org/files/48320/48320-0.txt')"
        )
    },
}
docs = await vs.asearch(query, search_type="similarity", metadata=filter, k=3)
for d in docs:
    print(d.page_content, " -> ", d.metadata, "\n====\n")
```
```output

在他的工作中拥有所有可能对他有用的知识，这是我努力做到的。如果我没记错的话，你在我们友谊的早期某个场合，非常准确地定义了我的限制。”

“是的，”我笑着回答道。“那是一份奇特的文件。我记得哲学、天文学和政治都被标记为零。植物学是可变的，地质学则对任何地区的泥土污渍非常深刻。”  ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 

====

当我入睡时，他坐在那里，阳光照在他坚毅的鹰钩鼻特征上。当我被突然的感叹声吵醒时，他还是坐在那里，我发现夏日的阳光照进了房间。烟斗仍然夹在他的嘴里，烟雾仍然升腾，房间里弥漫着浓浓的烟草雾气，但是我在前一天晚上看到的一堆烟叶已经不见了。

“醒了，华生？”他问道。

“是的。”

“愿意去开个早晨的车吗？”  ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 

====

“我瞥了一眼桌子上的书，尽管我不懂德语，但我可以看到其中两本是科学著作，其他的是诗集。然后我走到窗前，希望能看到乡间的一些景色，但是一扇厚重的橡木百叶窗挡住了视线。这是一所非常安静的房子。走廊里有一只古老的时钟在响个不停，但其他一切都死一般的寂静。我有一种模糊的感觉  ->  {'source': 'https://www.gutenberg.org/files/48320/48320-0.txt'} 

====