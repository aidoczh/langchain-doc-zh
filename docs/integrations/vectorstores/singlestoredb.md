# SingleStoreDB

[SingleStoreDB](https://singlestore.com/) 是一个强大的、高性能的分布式 SQL 数据库解决方案，旨在在[云端](https://www.singlestore.com/cloud/)和本地环境中均表现出色。凭借多功能的功能集，它提供了无缝部署选项，同时提供无与伦比的性能。

SingleStoreDB 的一个显著特点是其对向量存储和操作的高级支持，使其成为需要复杂 AI 能力（如文本相似性匹配）的应用程序的理想选择。借助内置的向量函数，如[点积](https://docs.singlestore.com/managed-service/en/reference/sql-reference/vector-functions/dot_product.html)和[欧几里得距离](https://docs.singlestore.com/managed-service/en/reference/sql-reference/vector-functions/euclidean_distance.html)，SingleStoreDB 赋予开发人员有效实现复杂算法的能力。

对于希望在 SingleStoreDB 中利用向量数据的开发人员，提供了一份全面的教程，指导他们[处理向量数据](https://docs.singlestore.com/managed-service/en/developer-resources/functional-extensions/working-with-vector-data.html)的复杂性。该教程深入探讨了 SingleStoreDB 中的向量存储，展示了其根据向量相似性进行搜索的能力。利用向量索引，查询可以以非凡的速度执行，实现快速检索相关数据。

此外，SingleStoreDB 的向量存储与基于 Lucene 的[全文索引](https://docs.singlestore.com/cloud/developer-resources/functional-extensions/working-with-full-text-search/)完美集成，实现强大的文本相似性搜索。用户可以根据文档元数据对象的选定字段过滤搜索结果，提高查询精度。

SingleStoreDB 的独特之处在于其能够以各种方式结合向量和全文搜索，提供灵活性和多样性。无论是通过文本或向量相似性进行预过滤并选择最相关数据，还是采用加权和方法计算最终相似度分数，开发人员都有多种选择。

总的来说，SingleStoreDB 为管理和查询向量数据提供了全面的解决方案，为基于 AI 的应用程序提供了无与伦比的性能和灵活性。

```python
# 通过 singlestoredb Python 连接器方便地建立与数据库的连接。
# 请确保该连接器已安装在您的工作环境中。
%pip install --upgrade --quiet  singlestoredb
```

```python
import getpass
import os
# 我们想使用 OpenAIEmbeddings，因此必须获取 OpenAI API 密钥。
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
from langchain_community.vectorstores import SingleStoreDB
from langchain_community.vectorstores.utils import DistanceStrategy
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
```

```python
# 加载文档
# 我们将为此示例使用一些人工数据
docs = [
    Document(
        page_content="""在干旱的沙漠中，一场突如其来的暴雨带来了缓解，
            雨滴跳舞在干渴的大地上，用芬芳的土壤气息使景观恢复生机。
            """,
        metadata={"category": "rain"},
    ),
    Document(
        page_content="""在繁华的城市景观中，雨无情地倾泻而下，
            在人行道上营造出淅淅沥沥的交响曲，而雨伞
            如同五彩缤纷的花朵在灰色的海洋中绽放。
            """,
        metadata={"category": "rain"},
    ),
    Document(
        page_content="""在高山之巅，雨变成了一种细腻的
            薄雾，将峰顶包裹在神秘的面纱中，每一滴似乎
            向下述说古老的岩石秘密。
            """,
        metadata={"category": "rain"},
    ),
    Document(
        page_content="""在乡村铺满一层柔软的
            雪花，雪落下，将世界包裹在宁静的寂静中
            如同自然的蕾丝般轻柔地落在树枝上。
            """,
        metadata={"category": "snow"},
    ),
    Document(
        page_content="""在城市景观中，雪花飘落，
            将繁忙的街道变成冬季仙境，孩子们的笑声
            在雪球和节日灯光的喧嚣中回荡。
            """,
        metadata={"category": "snow"},
    ),
    Document(
        page_content="""在崎岖的山峰之上，雪花以不屈不挠的
            强度落下，将景观雕刻成一片原始的高山乐园，
            冰冻的晶体在月光下闪烁，投下
            魔法般的魅力覆盖下方的荒野。
            """,
        metadata={"category": "snow"},
    ),
]
embeddings = OpenAIEmbeddings()
```

有几种方法可以建立与数据库的 [连接](https://singlestoredb-python.labs.singlestore.com/generated/singlestoredb.connect.html)。您可以设置环境变量，也可以向 `SingleStoreDB constructor` 传递命名参数。另外，您还可以将这些参数提供给 `from_documents` 和 `from_texts` 方法。

```python
# 将连接 URL 设置为环境变量
os.environ["SINGLESTOREDB_URL"] = "root:pass@localhost:3306/db"
# 将文档加载到存储中
docsearch = SingleStoreDB.from_documents(
    docs,
    embeddings,
    table_name="notebook",  # 使用自定义名称的表
)
```

```python
query = "雪中的树"
docs = docsearch.similarity_search(query)  # 查找与查询相对应的文档
print(docs[0].page_content)
```

SingleStoreDB 通过基于元数据字段的预过滤功能提升了搜索能力。这一功能赋予开发人员和数据分析师调整查询的能力，确保搜索结果精确地符合其需求。通过使用特定的元数据属性来过滤搜索结果，用户可以缩小查询范围，仅关注相关的数据子集。

```python
query = "树枝"
docs = docsearch.similarity_search(
    query, filter={"category": "雪"}
)  # 查找与查询相对应且类别为"雪"的文档
print(docs[0].page_content)
```

通过利用 [ANN 向量索引](https://docs.singlestore.com/cloud/reference/sql-reference/vector-functions/vector-indexing/)，您可以在 SingleStore DB 8.5 版本或更高版本中提高搜索效率。在创建向量存储对象时，通过设置 `use_vector_index=True`，您可以激活此功能。此外，如果您的向量与默认的 OpenAI 嵌入尺寸 1536 不同，则确保相应地指定 `vector_size` 参数。

SingleStoreDB 提供了多种搜索策略，每种策略都经过精心设计，以满足特定的用例和用户偏好。默认的 `VECTOR_ONLY` 策略利用向量操作（如 `dot_product` 或 `euclidean_distance`）直接计算向量之间的相似度分数，而 `TEXT_ONLY` 则采用基于 Lucene 的全文搜索，特别适用于以文本为中心的应用。对于寻求平衡方法的用户，`FILTER_BY_TEXT` 首先根据文本相似性细化结果，然后进行向量比较，而 `FILTER_BY_VECTOR` 则优先考虑向量相似性，在评估文本相似性之前过滤结果以获得最佳匹配。值得注意的是，`FILTER_BY_TEXT` 和 `FILTER_BY_VECTOR` 都需要全文索引才能运行。此外，`WEIGHTED_SUM` 是一种复杂的策略，通过加权向量和文本相似度来计算最终的相似度分数，尽管它仅利用 `dot_product` 距离计算，并且还需要全文索引。这些多功能的策略使用户能够根据其独特的需求微调搜索，促进高效和精确的数据检索和分析。此外，SingleStoreDB 的混合方法（例如 `FILTER_BY_TEXT`、`FILTER_BY_VECTOR` 和 `WEIGHTED_SUM` 策略）无缝地结合了向量和基于文本的搜索，以最大限度地提高效率和准确性，确保用户可以充分利用平台的功能来满足各种应用的需求。

```python
docsearch = SingleStoreDB.from_documents(
    docs,
    embeddings,
    distance_strategy=DistanceStrategy.DOT_PRODUCT,  # 使用点积进行相似性搜索
    use_vector_index=True,  # 使用向量索引进行快速搜索
    use_full_text_search=True,  # 使用全文索引
)
vectorResults = docsearch.similarity_search(
    "干旱沙漠中的暴风雨, 雨",
    k=1,
    search_strategy=SingleStoreDB.SearchStrategy.VECTOR_ONLY,
    filter={"category": "雨"},
)
print(vectorResults[0].page_content)
textResults = docsearch.similarity_search(
    "干旱沙漠中的暴风雨, 雨",
    k=1,
    search_strategy=SingleStoreDB.SearchStrategy.TEXT_ONLY,
)
print(textResults[0].page_content)
filteredByTextResults = docsearch.similarity_search(
    "干旱沙漠中的暴风雨, 雨",
    k=1,
    search_strategy=SingleStoreDB.SearchStrategy.FILTER_BY_TEXT,
    filter_threshold=0.1,
)
print(filteredByTextResults[0].page_content)
filteredByVectorResults = docsearch.similarity_search(
    "干旱沙漠中的暴风雨, 雨",
    k=1,
    search_strategy=SingleStoreDB.SearchStrategy.FILTER_BY_VECTOR,
    filter_threshold=0.1,
)
print(filteredByVectorResults[0].page_content)
weightedSumResults = docsearch.similarity_search(
    "干旱沙漠中的暴风雨, 雨",
    k=1,
    search_strategy=SingleStoreDB.SearchStrategy.WEIGHTED_SUM,
    text_weight=0.2,
    vector_weight=0.8,
)
```

## 多模态示例：利用 CLIP 和 OpenClip 嵌入

在多模态数据分析领域，整合诸如图像和文本等多样信息类型变得日益关键。一种促进这种整合的强大工具是 [CLIP](https://openai.com/research/clip)，这是一种先进的模型，能够将图像和文本嵌入到共享语义空间中。通过这样做，CLIP 可以通过相似性搜索实现跨不同模态的相关内容检索。

为了说明，让我们考虑一个应用场景，我们旨在有效分析多模态数据。在这个示例中，我们利用 [OpenClip 多模态嵌入](/docs/integrations/text_embedding/open_clip)，这些嵌入利用了 CLIP 的框架。通过 OpenClip，我们可以无缝地嵌入文本描述和相应的图像，实现全面的分析和检索任务。无论是基于文本查询识别视觉上相似的图像，还是找到与特定视觉内容相关的相关文本段落，OpenClip 都赋予用户以非凡的效率和准确性从多模态数据中探索和提取见解。

```python
%pip install -U langchain openai singlestoredb langchain-experimental #（需要最新版本以支持多模态）
```

```python
import os
from langchain_community.vectorstores import SingleStoreDB
from langchain_experimental.open_clip import OpenCLIPEmbeddings
os.environ["SINGLESTOREDB_URL"] = "root:pass@localhost:3306/db"
TEST_IMAGES_DIR = "../../modules/images"
docsearch = SingleStoreDB(OpenCLIPEmbeddings())
image_uris = sorted(
    [
        os.path.join(TEST_IMAGES_DIR, image_name)
        for image_name in os.listdir(TEST_IMAGES_DIR)
        if image_name.endswith(".jpg")
    ]
)
# 添加图像
docsearch.add_images(uris=image_uris)
```