# RAGatouille

[RAGatouille](https://github.com/bclavie/RAGatouille) 让使用 ColBERT 变得非常简单！[ColBERT](https://github.com/stanford-futuredata/ColBERT) 是一个快速准确的检索模型，可以在数十毫秒内对大型文本集合进行可扩展的基于 BERT 的搜索。

我们可以通过多种方式使用 RAGatouille。

## 设置

集成位于 `ragatouille` 包中。

```bash
pip install -U ragatouille
```

```python
from ragatouille import RAGPretrainedModel
RAG = RAGPretrainedModel.from_pretrained("colbert-ir/colbertv2.0")
```

```output
[Jan 10, 10:53:28] 加载 segmented_maxsim_cpp 扩展 (设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息)...
``````output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/cuda/amp/grad_scaler.py:125: UserWarning: torch.cuda.amp.GradScaler 已启用，但 CUDA 不可用。禁用。
  warnings.warn(
```

## 检索器

我们可以将 RAGatouille 用作检索器。有关详细信息，请参见[RAGatouille 检索器](/docs/integrations/retrievers/ragatouille)

## 文档压缩器

我们还可以直接使用 RAGatouille 作为重新排序器。这将允许我们使用 ColBERT 对来自任何通用检索器的检索结果进行重新排序。这样做的好处是我们可以在任何现有索引之上进行，而不需要创建新的索引。我们可以通过在 LangChain 中使用[文档压缩器](/docs/how_to/contextual_compression)抽象来实现这一点。

## 设置原始检索器

首先，让我们设置一个原始检索器作为示例。

```python
import requests
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
def get_wikipedia_page(title: str):
    """
    检索维基百科页面的完整文本内容。
    :param title: str - 维基百科页面的标题。
    :return: str - 页面的完整文本内容。
    """
    # 维基百科 API 端点
    URL = "https://en.wikipedia.org/w/api.php"
    # API 请求的参数
    params = {
        "action": "query",
        "format": "json",
        "titles": title,
        "prop": "extracts",
        "explaintext": True,
    }
    # 自定义 User-Agent 标头以符合维基百科的最佳实践
    headers = {"User-Agent": "RAGatouille_tutorial/0.0.1 (ben@clavie.eu)"}
    response = requests.get(URL, params=params, headers=headers)
    data = response.json()
    # 提取页面内容
    page = next(iter(data["query"]["pages"].values()))
    return page["extract"] if "extract" in page else None
text = get_wikipedia_page("Hayao_Miyazaki")
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
texts = text_splitter.create_documents([text])
```

```python
retriever = FAISS.from_documents(texts, OpenAIEmbeddings()).as_retriever(
    search_kwargs={"k": 10}
)
```

```python
docs = retriever.invoke("宫崎骏创办了哪个动画工作室")
docs[0]
```

```output
Document(page_content='合作项目。1984年4月，宫崎在杉並区开设了自己的办公室，取名为Nibariki。')
```

我们可以看到结果与所提问的问题不太相关

## 使用 ColBERT 作为重新排序器

```python
from langchain.retrievers import ContextualCompressionRetriever
compression_retriever = ContextualCompressionRetriever(
    base_compressor=RAG.as_langchain_document_compressor(), base_retriever=retriever
)
compressed_docs = compression_retriever.invoke(
    "宫崎骏创办了哪个动画工作室"
)
```

```output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/amp/autocast_mode.py:250: UserWarning: 用户提供的设备类型为 'cuda'，但 CUDA 不可用。禁用。
  warnings.warn(
```

```python
compressed_docs[0]
```

```output
Document(page_content='1985年6月，宫崎、高畑、德间和铃木共同创办了动画制作公司吉卜力工作室，获得德间书店的资金支持。吉卜力工作室的第一部电影《天空之城》（1986年）聘请了《风之谷》的同一制作班底。宫崎为该片的设定设计灵感来自希腊建筑和“欧洲城市模板”。电影中的一些建筑也受到威尔士一个矿业城镇的启发；宫崎在首次到访时目睹了矿工罢工', metadata={'relevance_score': 26.5194149017334})
```

这个答案更相关！