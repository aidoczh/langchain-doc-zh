# Aerospike

[Aerospike Vector Search](https://aerospike.com/docs/vector) (AVS) 是 Aerospike 数据库的扩展，它可以在 Aerospike 中存储的大型数据集上进行搜索。这项新服务位于 Aerospike 之外，并构建了一个索引来执行这些搜索。

这个笔记本展示了 LangChain Aerospike VectorStore 集成的功能。

## 安装 AVS

在使用这个笔记本之前，我们需要运行一个 AVS 实例。可以使用[可用的安装方法](https://aerospike.com/docs/vector/install)之一。

完成后，将您的 AVS 实例的 IP 地址和端口存储起来，以便稍后在这个演示中使用：

```python
PROXIMUS_HOST = "<avs-ip>"
PROXIMUS_PORT = 5000
```

## 安装依赖

`sentence-transformers` 依赖项很大。这一步可能需要几分钟才能完成。

```python
!pip install --upgrade --quiet aerospike-vector-search==0.6.1 sentence-transformers langchain
```

## 下载引语数据集

我们将下载一个包含大约 100,000 条引语的数据集，并使用其中的一个子集进行语义搜索。

```python
!wget https://github.com/aerospike/aerospike-vector-search-examples/raw/7dfab0fccca0852a511c6803aba46578729694b5/quote-semantic-search/container-volumes/quote-search/data/quotes.csv.tgz
```
```output
--2024-05-10 17:28:17--  https://github.com/aerospike/aerospike-vector-search-examples/raw/7dfab0fccca0852a511c6803aba46578729694b5/quote-semantic-search/container-volumes/quote-search/data/quotes.csv.tgz
Resolving github.com (github.com)... 140.82.116.4
Connecting to github.com (github.com)|140.82.116.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://raw.githubusercontent.com/aerospike/aerospike-vector-search-examples/7dfab0fccca0852a511c6803aba46578729694b5/quote-semantic-search/container-volumes/quote-search/data/quotes.csv.tgz [following]
--2024-05-10 17:28:17--  https://raw.githubusercontent.com/aerospike/aerospike-vector-search-examples/7dfab0fccca0852a511c6803aba46578729694b5/quote-semantic-search/container-volumes/quote-search/data/quotes.csv.tgz
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.110.133, 185.199.109.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 11597643 (11M) [application/octet-stream]
Saving to: ‘quotes.csv.tgz’
quotes.csv.tgz      100%[===================>]  11.06M  1.94MB/s    in 6.1s
2024-05-10 17:28:23 (1.81 MB/s) - ‘quotes.csv.tgz’ saved [11597643/11597643]
```

## 将引语加载到文档中

我们将使用 `CSVLoader` 文档加载器加载我们的引语数据集。在这种情况下，`lazy_load` 返回一个迭代器，以更有效地摄取我们的引语。在这个例子中，我们只加载了 5,000 条引语。

```python
import itertools
import os
import tarfile
from langchain_community.document_loaders.csv_loader import CSVLoader
filename = "./quotes.csv"
if not os.path.exists(filename) and os.path.exists(filename + ".tgz"):
    # Untar the file
    with tarfile.open(filename + ".tgz", "r:gz") as tar:
        tar.extractall(path=os.path.dirname(filename))
NUM_QUOTES = 5000
documents = CSVLoader(filename, metadata_columns=["author", "category"]).lazy_load()
documents = list(
    itertools.islice(documents, NUM_QUOTES)
)  # Allows us to slice an iterator
```
```python
print(documents[0])
```
```output
page_content="quote: I'm selfish, impatient and a little insecure. I make mistakes, I am out of control and at times hard to handle. But if you can't handle me at my worst, then you sure as hell don't deserve me at my best." metadata={'source': './quotes.csv', 'row': 0, 'author': 'Marilyn Monroe', 'category': 'attributed-no-source, best, life, love, mistakes, out-of-control, truth, worst'}
```

## 创建您的嵌入器

在这一步中，我们使用 HuggingFaceEmbeddings 和 "all-MiniLM-L6-v2" 句子转换模型来嵌入我们的文档，以便进行向量搜索。

```python
from aerospike_vector_search.types import VectorDistanceMetric
from langchain_community.embeddings import HuggingFaceEmbeddings
MODEL_DIM = 384
MODEL_DISTANCE_CALC = VectorDistanceMetric.COSINE
embedder = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
```
```output
modules.json:   0%|          | 0.00/349 [00:00<?, ?B/s]
config_sentence_transformers.json:   0%|          | 0.00/116 [00:00<?, ?B/s]
README.md:   0%|          | 0.00/10.7k [00:00<?, ?B/s]
sentence_bert_config.json:   0%|          | 0.00/53.0 [00:00<?, ?B/s]
/opt/conda/lib/python3.11/site-packages/huggingface_hub/file_download.py:1132: FutureWarning: `resume_download` is deprecated and will be removed in version 1.0.0. Downloads always resume when possible. If you want to force a new download, use `force_download=True`.
  warnings.warn(
```
```output
config.json:   0%|          | 0.00/612 [00:00<?, ?B/s]
```
```output
/opt/conda/lib/python3.11/site-packages/huggingface_hub/file_download.py:1132: 未来警告: `resume_download` 已弃用，并将在 1.0.0 版本中删除。在可能的情况下，下载总是会恢复。如果要强制进行新下载，请使用 `force_download=True`。
  warnings.warn(
```
```output
model.safetensors:   0%|          | 0.00/90.9M [00:00<?, ?B/s]
```
```output
tokenizer_config.json:   0%|          | 0.00/350 [00:00<?, ?B/s]
```
```output
vocab.txt:   0%|          | 0.00/232k [00:00<?, ?B/s]
```
```output
tokenizer.json:   0%|          | 0.00/466k [00:00<?, ?B/s]
```
```output
special_tokens_map.json:   0%|          | 0.00/112 [00:00<?, ?B/s]
```
```output
1_Pooling/config.json:   0%|          | 0.00/190 [00:00<?, ?B/s]
```

## 创建 Aerospike 索引并嵌入文档

在添加文档之前，我们需要在 Aerospike 数据库中创建一个索引。在下面的示例中，我们使用一些便利代码来检查预期的索引是否已经存在。

```python
from aerospike_vector_search import AdminClient, Client, HostPort
from aerospike_vector_search.types import VectorDistanceMetric
from langchain_community.vectorstores import Aerospike
# 这里我们使用了之前配置的 AVS 主机和端口
seed = HostPort(host=PROXIMUS_HOST, port=PROXIMUS_PORT)
# 存放我们向量的命名空间。这应该与您的 docstore.conf 文件中配置的向量匹配。
NAMESPACE = "test"
# 我们新索引的名称。
INDEX_NAME = "quote-miniLM-L6-v2"
# 在创建索引和插入文档时，AVS 需要知道哪个元数据键包含我们的向量。
VECTOR_KEY = "vector"
client = Client(seeds=seed)
admin_client = AdminClient(
    seeds=seed,
)
index_exists = False
# 检查索引是否已经存在。如果不存在，则创建它
for index in admin_client.index_list():
    if index["id"]["namespace"] == NAMESPACE and index["id"]["name"] == INDEX_NAME:
        index_exists = True
        print(f"{INDEX_NAME} 已经存在。跳过创建")
        break
if not index_exists:
    print(f"{INDEX_NAME} 不存在。创建索引")
    admin_client.index_create(
        namespace=NAMESPACE,
        name=INDEX_NAME,
        vector_field=VECTOR_KEY,
        vector_distance_metric=MODEL_DISTANCE_CALC,
        dimensions=MODEL_DIM,
        index_meta_data={
            "model": "miniLM-L6-v2",
            "date": "05/04/2024",
            "dim": str(MODEL_DIM),
            "distance": "cosine",
        },
    )
admin_client.close()
docstore = Aerospike.from_documents(
    documents,
    embedder,
    client=client,
    namespace=NAMESPACE,
    vector_key=VECTOR_KEY,
    index_name=INDEX_NAME,
    distance_strategy=MODEL_DISTANCE_CALC,
)
```
```output
quote-miniLM-L6-v2 不存在。创建索引
```

## 搜索文档

现在我们已经嵌入了我们的向量，我们可以在我们的引用上使用向量搜索。

```python
query = "有关宇宙之美的引用"
docs = docstore.similarity_search(
    query, k=5, index_name=INDEX_NAME, metadata_keys=["_id", "author"]
)
def print_documents(docs):
    for i, doc in enumerate(docs):
        print("~~~~ 文档", i, "~~~~")
        print("自动生成的 id:", doc.metadata["_id"])
        print("作者: ", doc.metadata["author"])
        print(doc.page_content)
        print("~~~~~~~~~~~~~~~~~~~~\n")
print_documents(docs)
```
```output
~~~~ 文档 0 ~~~~
自动生成的 id: f53589dd-e3e0-4f55-8214-766ca8dc082f
作者:  Carl Sagan, Cosmos
引用: 宇宙就是所有的存在，曾经存在和将来存在。我们对宇宙的最微弱的思考会激发我们 -- 脊椎会发麻，声音会突然停顿，有一种微弱的感觉，仿佛是远方的记忆，从高处坠落。我们知道我们正在接近最大的奥秘。
~~~~~~~~~~~~~~~~~~~~
~~~~ 文档 1 ~~~~
自动生成的 id: dde3e5d1-30b7-47b4-aab7-e319d14e1810
作者:  Elizabeth Gilbert
引用: 那爱，驱动着太阳和其他星球。
~~~~~~~~~~~~~~~~~~~~
~~~~ 文档 2 ~~~~
自动生成的 id: fd56575b-2091-45e7-91c1-9efff2fe5359
作者:  Renee Ahdieh, The Rose & the Dagger
引用: 从星星到星星。
~~~~~~~~~~~~~~~~~~~~
~~~~ 文档 3 ~~~~
自动生成的 id: 8567ed4e-885b-44a7-b993-e0caf422b3c9
作者:  Dante Alighieri, Paradiso
引用: 爱，驱动着太阳和其他星球
~~~~~~~~~~~~~~~~~~~~
~~~~ 文档 4 ~~~~
自动生成的 id: f868c25e-c54d-48cd-a5a8-14bf402f9ea8
作者:  Thich Nhat Hanh, Teachings on Love
## 将额外的引用作为文本嵌入
我们可以使用`add_texts`来添加额外的引用。
```python
docstore = Aerospike(
    client,
    embedder,
    NAMESPACE,
    index_name=INDEX_NAME,
    vector_key=VECTOR_KEY,
    distance_strategy=MODEL_DISTANCE_CALC,
)
ids = docstore.add_texts(
    [
        "quote: Rebellions are built on hope.",
        "quote: Logic is the beginning of wisdom, not the end.",
        "quote: If wishes were fishes, we’d all cast nets.",
    ],
    metadatas=[
        {"author": "Jyn Erso, Rogue One"},
        {"author": "Spock, Star Trek"},
        {"author": "Frank Herbert, Dune"},
    ],
)
print("New IDs")
print(ids)
```
```output

New IDs

['972846bd-87ae-493b-8ba3-a3d023c03948', '8171122e-cbda-4eb7-a711-6625b120893b', '53b54409-ac19-4d90-b518-d7c40bf5ee5d']

```
## 使用最大边际相关性搜索来搜索文档
我们可以使用最大边际相关性搜索来找到与我们的查询相似但彼此不同的向量。在这个例子中，我们使用`as_retriever`创建一个检索器对象，但也可以直接调用`docstore.max_marginal_relevance_search`来完成。`lambda_mult`搜索参数确定了我们查询响应的多样性。0对应于最大多样性，1对应于最小多样性。
```python
query = "A quote about our favorite four-legged pets"
retriever = docstore.as_retriever(
    search_type="mmr", search_kwargs={"fetch_k": 20, "lambda_mult": 0.7}
)
matched_docs = retriever.invoke(query)
print_documents(matched_docs)
```
```output

~~~~ Document 0 ~~~~

auto-generated id: 67d5b23f-b2d2-4872-80ad-5834ea08aa64

author:  John Grogan, Marley and Me: Life and Love With the World's Worst Dog

quote: Such short little lives our pets have to spend with us, and they spend most of it waiting for us to come home each day. It is amazing how much love and laughter they bring into our lives and even how much closer we become with each other because of them.

~~~~~~~~~~~~~~~~~~~~

~~~~ Document 1 ~~~~

auto-generated id: a9b28eb0-a21c-45bf-9e60-ab2b80e988d8

author:  John Grogan, Marley and Me: Life and Love With the World's Worst Dog

quote: Dogs are great. Bad dogs, if you can really call them that, are perhaps the greatest of them all.

~~~~~~~~~~~~~~~~~~~~

~~~~ Document 2 ~~~~

auto-generated id: ee7434c8-2551-4651-8a22-58514980fb4a

author:  Colleen Houck, Tiger's Curse

quote: He then put both hands on the door on either side of my head and leaned in close, pinning me against it. I trembled like a downy rabbit caught in the clutches of a wolf. The wolf came closer. He bent his head and began nuzzling my cheek. The problem was…I wanted the wolf to devour me.

~~~~~~~~~~~~~~~~~~~~

~~~~ Document 3 ~~~~

auto-generated id: 9170804c-a155-473b-ab93-8a561dd48f91

author:  Ray Bradbury

quote: Stuff your eyes with wonder," he said, "live as if you'd drop dead in ten seconds. See the world. It's more fantastic than any dream made or paid for in factories. Ask no guarantees, ask for no security, there never was such an animal. And if there were, it would be related to the great sloth which hangs upside down in a tree all day every day, sleeping its life away. To hell with that," he said, "shake the tree and knock the great sloth down on his ass.

~~~~~~~~~~~~~~~~~~~~

```
## 使用相关阈值搜索文档
另一个有用的功能是使用相关阈值进行相似性搜索。通常，我们只想要与我们的查询最相似的结果，但也在某个接近范围内。相关性为1表示最相似，相关性为0表示最不相似。
```python
query = "A quote about stormy weather"
retriever = docstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={
        "score_threshold": 0.4
    },  # A greater value returns items with more relevance
)
matched_docs = retriever.invoke(query)
print_documents(matched_docs)
```
```output

~~~~ Document 0 ~~~~

auto-generated id: 2c1d6ee1-b742-45ea-bed6-24a1f655c849

author:  Roy T. Bennett, The Light in the Heart

quote: Never lose hope. Storms make people stronger and never last forever.

~~~~~~~~~~~~~~~~~~~~

~~~~ Document 1 ~~~~

auto-generated id: 5962c2cf-ffb5-4e03-9257-bdd630b5c7e9

author:  Roy T. Bennett, The Light in the Heart

quote: Storms don't come to teach us painful lessons, rather they were meant to wash us clean.

~~~~~~~~~~~~~~~~~~~~

```
引言：困难和逆境狠狠地施加在我们身上，导致我们支离破碎，但它们是个人成长的必要元素，揭示了我们真正的潜力。我们必须忍受并克服它们，继续前行。永远不要失去希望。风暴让人变得更强大，但终究不会永远持续。
~~~~ 文档 2 ~~~~
自动生成的 ID：3bbcc4ca-de89-4196-9a46-190a50bf6c47
作者：文森特·梵高，《文森特·梵高的信件》
引言：即使在风暴中也有平静。
~~~~ 文档 3 ~~~~
自动生成的 ID：37d8cf02-fc2f-429d-b2b6-260a05286108
作者：埃德温·摩根，《生命之书》
引言：情人节的天气，用雨吻我眼睛上的睫毛，来吧，让我们一起摇摆，在树下，雷声都去死吧。
## 清理
我们需要确保关闭客户端以释放资源并清理线程。
```python
client.close()
```

## 准备、开始、搜索！

现在您已经了解了Aerospike Vector Search的LangChain集成，您拥有Aerospike数据库和LangChain生态系统的强大功能。祝您建设愉快！