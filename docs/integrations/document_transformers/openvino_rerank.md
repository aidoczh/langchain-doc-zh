# OpenVINO Reranker

[OpenVINO™](https://github.com/openvinotoolkit/openvino) 是一个用于优化和部署 AI 推断的开源工具包。OpenVINO™ Runtime 支持多种硬件[设备](https://github.com/openvinotoolkit/openvino?tab=readme-ov-file#supported-hardware-matrix)，包括 x86 和 ARM CPU，以及 Intel GPU。它可以帮助提升计算机视觉、自动语音识别、自然语言处理和其他常见任务中的深度学习性能。

Hugging Face rerank 模型可以通过 ``OpenVINOReranker`` 类来支持 OpenVINO。如果你有 Intel GPU，你可以指定 `model_kwargs={"device": "GPU"}` 来在其上运行推断。

```python
%pip install --upgrade-strategy eager "optimum[openvino,nncf]" --quiet
%pip install --upgrade --quiet  faiss-cpu
```

```python
# 用于打印文档的辅助函数
def pretty_print_docs(docs):
    print(
        f"\n{'-' * 100}\n".join(
            [
                f"Document {i+1}:\n\n{d.page_content}\nMetadata: {d.metadata}"
                for i, d in enumerate(docs)
            ]
        )
    )
```

## 设置基础向量存储检索器

让我们首先初始化一个简单的向量存储检索器，并存储 2023 年的国情咨文演讲（以块形式）。我们可以设置检索器来检索大量（20 个）文档。

```python
from langchain.embeddings import OpenVINOEmbeddings
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_text_splitters import RecursiveCharacterTextSplitter
documents = TextLoader(
    "../../how_to/state_of_the_union.txt",
).load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=100)
texts = text_splitter.split_documents(documents)
for idx, text in enumerate(texts):
    text.metadata["id"] = idx
embedding = OpenVINOEmbeddings(
    model_name_or_path="sentence-transformers/all-mpnet-base-v2"
)
retriever = FAISS.from_documents(texts, embedding).as_retriever(search_kwargs={"k": 20})
query = "What did the president say about Ketanji Brown Jackson"
docs = retriever.invoke(query)
pretty_print_docs(docs)
```

## 使用 OpenVINO 进行重新排名

现在让我们用 `ContextualCompressionRetriever` 将基础检索器包装起来，使用 `OpenVINOReranker` 作为压缩器。

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain_community.document_compressors.openvino_rerank import OpenVINOReranker
model_name = "BAAI/bge-reranker-large"
ov_compressor = OpenVINOReranker(model_name_or_path=model_name, top_n=4)
compression_retriever = ContextualCompressionRetriever(
    base_compressor=ov_compressor, base_retriever=retriever
)
compressed_docs = compression_retriever.invoke(
    "What did the president say about Ketanji Jackson Brown"
)
print([doc.metadata["id"] for doc in compressed_docs])
```

重新排名后，前 4 个文档与基础检索器检索到的前 4 个文档不同。

```python
pretty_print_docs(compressed_docs)
```

```output
文档 1：
美国总统最严肃的宪法责任之一是提名某人担任美国最高法院的法官。
4 天前，我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律专家，将延续布雷尔大法官的卓越传统。
元数据：{'id': 0, 'relevance_score': tensor(0.6148)}
----------------------------------------------------------------------------------------------------
文档 2：
他永远不会扼杀他们对自由的热爱。他永远不会削弱自由世界的决心。
我们今晚聚集在一个经历了这个国家有史以来最艰难两年的美国。
这场大流行病是惩罚性的。
许多家庭靠发工资维持生计，努力跟上食品、汽油、住房等成本的上涨。我理解。
元数据：{'id': 16, 'relevance_score': tensor(0.0373)}
----------------------------------------------------------------------------------------------------
文档 3：
曾在私人执业中担任高级诉讼律师。曾担任联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。一个达成共识的人。自她被提名以来，她得到了广泛的支持，从警察兄弟会到民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民制度。
元数据：{'id': 18, 'relevance_score': tensor(0.0131)}
----------------------------------------------------------------------------------------------------
文档 4：
对所有美国人，我会如我一直承诺的那样诚实。一个俄罗斯独裁者入侵外国，给世界带来了代价。
我正在采取有力措施，确保我们的制裁之痛针对俄罗斯的经济。我将利用我们掌握的一切工具来保护美国的企业和消费者。
今晚，我可以宣布，美国已与其他 30 个国家合作，从世界各地的储备释放了 6 千万桶石油。
元数据：{'id': 6, 'relevance_score': tensor(0.0098)}
```

## 导出 IR 模型

可以使用 ``OVModelForSequenceClassification`` 将重新排名模型导出为 OpenVINO IR 格式，并从本地文件夹加载模型。

```python
from pathlib import Path
ov_model_dir = "bge-reranker-large-ov"
if not Path(ov_model_dir).exists():
    ov_compressor.save_model(ov_model_dir)
```

```python
ov_compressor = OpenVINOReranker(model_name_or_path=ov_model_dir)
```

```output
正在将模型编译为 CPU...
```

更多信息请参考：

* [OpenVINO LLM 指南](https://docs.openvino.ai/2024/learn-openvino/llm_inference_guide.html)。

* [OpenVINO 文档](https://docs.openvino.ai/2024/home.html)。

* [OpenVINO 入门指南](https://www.intel.com/content/www/us/en/content-details/819067/openvino-get-started-guide.html)。

* [LangChain 的 RAG Notebook](https://github.com/openvinotoolkit/openvino_notebooks/tree/latest/notebooks/llm-rag-langchain)。