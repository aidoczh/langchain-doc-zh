# 交叉编码器重排序器

本文介绍了如何在检索器中使用自己的交叉编码器实现重排序，该交叉编码器来自[Hugging Face交叉编码器模型](https://huggingface.co/cross-encoder)或实现交叉编码器功能的Hugging Face模型（例如：BAAI/bge-reranker-base）。“SagemakerEndpointCrossEncoder”使您能够在Sagemaker上使用这些加载的HuggingFace模型。

这是在[ContextualCompressionRetriever](/docs/how_to/contextual_compression)中的想法基础上构建的。本文档的整体结构来自[Cohere Reranker文档](/docs/integrations/retrievers/cohere-reranker)。

有关为什么可以将交叉编码器与嵌入一起用于更好的检索的重排序机制的更多信息，请参阅[Hugging Face交叉编码器文档](https://www.sbert.net/examples/applications/cross-encoder/README.html)。

```python
#!pip install faiss sentence_transformers
# 或者（根据Python版本）
#!pip install faiss-cpu sentence_transformers
```

```python
# 用于打印文档的辅助函数
def pretty_print_docs(docs):
    print(
        f"\n{'-' * 100}\n".join(
            [f"文档 {i+1}:\n\n" + d.page_content for i, d in enumerate(docs)]
        )
    )
```

## 设置基础向量存储检索器

让我们首先初始化一个简单的向量存储检索器，并存储2023年的国情咨文演讲（以块为单位）。我们可以设置检索器以检索高数量（20）的文档。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
documents = TextLoader("../../how_to/state_of_the_union.txt").load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=100)
texts = text_splitter.split_documents(documents)
embeddingsModel = HuggingFaceEmbeddings(
    model_name="sentence-transformers/msmarco-distilbert-dot-v5"
)
retriever = FAISS.from_documents(texts, embeddingsModel).as_retriever(
    search_kwargs={"k": 20}
)
query = "经济计划是什么？"
docs = retriever.invoke(query)
pretty_print_docs(docs)
```

## 使用CrossEncoderReranker进行重排序

现在让我们用`ContextualCompressionRetriever`包装我们的基础检索器。`CrossEncoderReranker`使用`HuggingFaceCrossEncoder`对返回的结果进行重排序。

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import CrossEncoderReranker
from langchain_community.cross_encoders import HuggingFaceCrossEncoder
model = HuggingFaceCrossEncoder(model_name="BAAI/bge-reranker-base")
compressor = CrossEncoderReranker(model=model, top_n=3)
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor, base_retriever=retriever
)
compressed_docs = compression_retriever.invoke("经济计划是什么？")
pretty_print_docs(compressed_docs)
```

```output
文档 1：
在美国进行更多的基础设施和创新。
在美国进行更多的商品运输，更快更便宜。
在美国提供更多能够获得良好收入的工作。
而不是依赖外国供应链，让我们在美国制造。
经济学家称之为“增加我们经济的生产能力”。
我称之为建设一个更好的美国。
我的抑制通胀计划将降低您的成本并降低赤字。
----------------------------------------------------------------------------------------------------
文档 2：
其次，通过应对气候变化，为家庭平均每年节约500美元的能源成本。
让我们为您的住宅和企业提供投资和税收抵免，以实现能源高效，您将获得税收抵免；将太阳能、风能等清洁能源产量翻倍；降低电动汽车价格，每月节省80美元，因为您再也不必在加油站支付费用。
----------------------------------------------------------------------------------------------------
文档 3：
看看汽车。
去年，没有足够的半导体来制造人们想要购买的所有汽车。
猜猜，汽车价格上涨了。
所以我们有一个选择。
对抗通胀的一种方法是降低工资，让美国人变穷。
我有一个更好的抑制通胀计划。
降低您的成本，而不是降低您的工资。
在美国制造更多汽车和半导体。
在美国进行更多的基础设施和创新。
在美国进行更多的商品运输，更快更便宜。
```

## 将Hugging Face模型上传到SageMaker端点

这是一个示例 `inference.py` 文件，用于创建一个与 `SagemakerEndpointCrossEncoder` 配合使用的端点。有关详细的逐步指导，请参考[这篇文章](https://huggingface.co/blog/kchoe/deploy-any-huggingface-model-to-sagemaker)。

它会动态下载 Hugging Face 模型，因此您无需在 `model.tar.gz` 中保留诸如 `pytorch_model.bin` 等模型工件。

```python
import json
import logging
from typing import List
import torch
from sagemaker_inference import encoder
from transformers import AutoModelForSequenceClassification, AutoTokenizer
PAIRS = "pairs"
SCORES = "scores"
class CrossEncoder:
    def __init__(self) -> None:
        self.device = (
            torch.device("cuda") if torch.cuda.is_available() else torch.device("cpu")
        )
        logging.info(f"Using device: {self.device}")
        model_name = "BAAI/bge-reranker-base"
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)
        self.model = AutoModelForSequenceClassification.from_pretrained(model_name)
        self.model = self.model.to(self.device)
    def __call__(self, pairs: List[List[str]]) -> List[float]:
        with torch.inference_mode():
            inputs = self.tokenizer(
                pairs,
                padding=True,
                truncation=True,
                return_tensors="pt",
                max_length=512,
            )
            inputs = inputs.to(self.device)
            scores = (
                self.model(**inputs, return_dict=True)
                .logits.view(
                    -1,
                )
                .float()
            )
        return scores.detach().cpu().tolist()
def model_fn(model_dir: str) -> CrossEncoder:
    try:
        return CrossEncoder()
    except Exception:
        logging.exception(f"Failed to load model from: {model_dir}")
        raise
def transform_fn(
    cross_encoder: CrossEncoder, input_data: bytes, content_type: str, accept: str
) -> bytes:
    payload = json.loads(input_data)
    model_output = cross_encoder(**payload)
    output = {SCORES: model_output}
    return encoder.encode(output, accept)
```

