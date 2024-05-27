# RAGatouille

[RAGatouille](https://github.com/bclavie/RAGatouille) 让使用 `ColBERT` 变得非常简单！

[ColBERT](https://github.com/stanford-futuredata/ColBERT) 是一个快速准确的检索模型，可以在几十毫秒内对大型文本集合进行可扩展的基于 BERT 的搜索。

我们可以将其用作[检索器](/docs/how_to#retrievers)。它将显示特定于此集成的功能。在完成后，探索[相关的用例页面](/docs/how_to#qa-with-rag)可能有助于了解如何将此向量存储作为更大链条的一部分使用。

本页面介绍了如何在 LangChain 链中使用[RAGatouille](https://github.com/bclavie/RAGatouille)作为检索器。

## 设置

该集成位于 `ragatouille` 包中。

```bash
pip install -U ragatouille
```

## 用法

以下示例摘自他们的文档

```python
from ragatouille import RAGPretrainedModel
RAG = RAGPretrainedModel.from_pretrained("colbert-ir/colbertv2.0")
```

```python
import requests
def get_wikipedia_page(title: str):
    """
    检索维基百科页面的完整文本内容。
    :param title: str - 维基百科页面的标题。
    :return: str - 页面的完整文本内容（原始字符串）。
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
    # 自定义用户代理标头，以符合维基百科的最佳实践
    headers = {"User-Agent": "RAGatouille_tutorial/0.0.1 (ben@clavie.eu)"}
    response = requests.get(URL, params=params, headers=headers)
    data = response.json()
    # 提取页面内容
    page = next(iter(data["query"]["pages"].values()))
    return page["extract"] if "extract" in page else None
```

```python
full_document = get_wikipedia_page("Hayao_Miyazaki")
```

```python
RAG.index(
    collection=[full_document],
    index_name="Miyazaki-123",
    max_document_length=180,
    split_documents=True,
)
```

```output
[Jan 07, 10:38:18] #> 创建目录 .ragatouille/colbert/indexes/Miyazaki-123 
#> 开始...
[Jan 07, 10:38:23] 加载 segmented_maxsim_cpp 扩展（设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息）...
```

```output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/cuda/amp/grad_scaler.py:125: UserWarning: torch.cuda.amp.GradScaler is enabled, but CUDA is not available.  Disabling.
  warnings.warn(
  0%|          | 0/2 [00:00<?, ?it/s]/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/amp/autocast_mode.py:250: UserWarning: User provided device_type of 'cuda', but CUDA is not available. Disabling
  warnings.warn(
```

```output
[Jan 07, 10:38:24] [0] 		 #> 编码 81 段落..
```

```output
 50%|█████     | 1/2 [00:02<00:02,  2.85s/it]/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/amp/autocast_mode.py:250: UserWarning: User provided device_type of 'cuda', but CUDA is not available. Disabling
  warnings.warn(
100%|██████████| 2/2 [00:03<00:00,  1.74s/it]
警告：将 10001 个点聚类到 1024 个质心：请提供至少 39936 个训练点
````

[一月 07, 10:38:27] [0] 		 avg_doclen_est = 129.9629669189453 	 len(local_sample) = 81

[一月 07, 10:38:27] [0] 		 创建 1,024 个分区。

[一月 07, 10:38:27] [0] 		 *预计* 10,527 个嵌入。

[一月 07, 10:38:27] [0] 		 #> 将索引计划保存到 .ragatouille/colbert/indexes/Miyazaki-123/plan.json ..

在 128D 中对 10001 个点进行聚类，重做 1 次，20 次迭代

  预处理时间 0.00 秒

  迭代 0 (0.02 秒，搜索 0.02 秒)：目标值=3772.41 不平衡度=1.562 分裂次数=0       

  迭代 1 (0.02 秒，搜索 0.02 秒)：目标值=2408.99 不平衡度=1.470 分裂次数=1       

  迭代 2 (0.03 秒，搜索 0.03 秒)：目标值=2243.87 不平衡度=1.450 分裂次数=0       

  迭代 3 (0.04 秒，搜索 0.04 秒)：目标值=2168.48 不平衡度=1.444 分裂次数=0       

  迭代 4 (0.05 秒，搜索 0.05 秒)：目标值=2134.26 不平衡度=1.449 分裂次数=0       

  迭代 5 (0.06 秒，搜索 0.05 秒)：目标值=2117.18 不平衡度=1.449 分裂次数=0       

  迭代 6 (0.06 秒，搜索 0.06 秒)：目标值=2108.43 不平衡度=1.449 分裂次数=0       

  迭代 7 (0.07 秒，搜索 0.07 秒)：目标值=2102.62 不平衡度=1.450 分裂次数=0       

  迭代 8 (0.08 秒，搜索 0.08 秒)：目标值=2100.68 不平衡度=1.451 分裂次数=0       

  迭代 9 (0.09 秒，搜索 0.08 秒)：目标值=2099.66 不平衡度=1.451 分裂次数=0       

  迭代 10 (0.10 秒，搜索 0.09 秒)：目标值=2099.03 不平衡度=1.451 分裂次数=0       

  迭代 11 (0.10 秒，搜索 0.10 秒)：目标值=2098.67 不平衡度=1.453 分裂次数=0       

  迭代 12 (0.11 秒，搜索 0.11 秒)：目标值=2097.78 不平衡度=1.455 分裂次数=0       

  迭代 13 (0.12 秒，搜索 0.12 秒)：目标值=2097.31 不平衡度=1.455 分裂次数=0       

  迭代 14 (0.13 秒，搜索 0.12 秒)：目标值=2097.13 不平衡度=1.455 分裂次数=0       

  迭代 15 (0.14 秒，搜索 0.13 秒)：目标值=2097.09 不平衡度=1.455 分裂次数=0       

  迭代 16 (0.14 秒，搜索 0.14 秒)：目标值=2097.09 不平衡度=1.455 分裂次数=0       

  迭代 17 (0.15 秒，搜索 0.15 秒)：目标值=2097.09 不平衡度=1.455 分裂次数=0       

  迭代 18 (0.16 秒，搜索 0.15 秒)：目标值=2097.09 不平衡度=1.455 分裂次数=0       

  迭代 19 (0.17 秒，搜索 0.16 秒)：目标值=2097.09 不平衡度=1.455 分裂次数=0       

[0.037, 0.038, 0.041, 0.036, 0.035, 0.036, 0.034, 0.036, 0.034, 0.034, 0.036, 0.037, 0.032, 0.039, 0.035, 0.039, 0.033, 0.035, 0.035, 0.037, 0.037, 0.037, 0.037, 0.037, 0.038, 0.034, 0.037, 0.035, 0.036, 0.037, 0.036, 0.04, 0.037, 0.037, 0.036, 0.034, 0.037, 0.035, 0.038, 0.039, 0.037, 0.039, 0.035, 0.036, 0.036, 0.035, 0.035, 0.038, 0.037, 0.033, 0.036, 0.032, 0.034, 0.035, 0.037, 0.037, 0.041, 0.037, 0.039, 0.033, 0.035, 0.033, 0.036, 0.036, 0.038, 0.036, 0.037, 0.038, 0.035, 0.035, 0.033, 0.034, 0.032, 0.038, 0.037, 0.037, 0.036, 0.04, 0.033, 0.037, 0.035, 0.04, 0.036, 0.04, 0.032, 0.037, 0.036, 0.037, 0.034, 0.042, 0.037, 0.035, 0.035, 0.038, 0.034, 0.036, 0.041, 0.035, 0.036, 0.037, 0.041, 0.04, 0.036, 0.036, 0.035, 0.036, 0.034, 0.033, 0.036, 0.033, 0.037, 0.038, 0.036, 0.033, 0.038, 0.037, 0.038, 0.037, 0.039, 0.04, 0.034, 0.034, 0.036, 0.039, 0.033, 0.037, 0.035, 0.037]

[一月 07, 10:38:27] [0] 		 #> 编码 81 个段落..

``````

```output
[Jan 07, 10:38:30] #> 优化 IVF 以存储从质心到 pid 列表的映射..
[Jan 07, 10:38:30] #> 构建 emb2pid 映射..
[Jan 07, 10:38:30] len(emb2pid) = 10527
[Jan 07, 10:38:30] #> 已将优化后的 IVF 保存到 .ragatouille/colbert/indexes/Miyazaki-123/ivf.pid.pt
#> 连接中...
索引完成！
```

```python
results = RAG.search(query="宫崎骏创立了哪家动画工作室？", k=3)
```

```output
首次加载索引 Miyazaki-123 的搜索器... 这可能需要几秒钟
[Jan 07, 10:38:34] 加载 segmented_maxsim_cpp 扩展 (设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息)...
[Jan 07, 10:38:35] #> 加载编解码器...
[Jan 07, 10:38:35] #> 加载 IVF...
[Jan 07, 10:38:35] 加载 segmented_lookup_cpp 扩展 (设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息)...
```

```output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/cuda/amp/grad_scaler.py:125: UserWarning: torch.cuda.amp.GradScaler is enabled, but CUDA is not available.  Disabling.
  warnings.warn(
```

```output
[Jan 07, 10:38:35] #> 加载 doclens...
```

```output
100%|███████████████████████████████████████████████████████| 1/1 [00:00<00:00, 3872.86it/s]
```

```output
[Jan 07, 10:38:35] #> 加载编码和残差...
```

```output
100%|████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 604.89it/s]
```

```output
[Jan 07, 10:38:35] 加载 filter_pids_cpp 扩展 (设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息)...
```

```output
[Jan 07, 10:38:35] 加载 decompress_residuals_cpp 扩展 (设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息)...
搜索器已加载！
#> QueryTokenizer.tensorize(batch_text[0], batch_background[0], bsize) ==
#> 输入: . 宫崎骏创立了哪家动画工作室？, 		 True, 		 None
#> 输出 IDs: torch.Size([32]), tensor([  101,     1,  2054,  7284,  2996,  2106,  2771,  3148, 18637,  2179,
         1029,   102,   103,   103,   103,   103,   103,   103,   103,   103,
          103,   103,   103,   103,   103,   103,   103,   103,   103,   103,
          103,   103])
#> 输出 Mask: torch.Size([32]), tensor([1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0])
```

```python
results
```

```output
[{'content': '1984年4月，宫崎骏在杉並区开设了自己的办公室，取名为Nibariki。\n\n\n=== 吉卜力工作室 ===\n\n\n==== 早期电影 (1985–1996) ====\n1985年6月，宫崎、高畑、德间和铃木创立了动画制作公司吉卜力工作室，获得了德间书店的资助。吉卜力工作室的第一部电影《天空之城》(1986) 使用了《风之谷》的同一制作团队。宫崎为电影的场景设计灵感来源于希腊建筑和“欧洲城市的模板”。',
  'score': 25.90749740600586,
  'rank': 1},
 {'content': '宫崎骏（宮崎 駿 or 宮﨑 駿, Miyazaki Hayao, [mijaꜜzaki hajao]; 生于1941年1月5日）是一位日本动画师、电影制作人和漫画家。作为吉卜力工作室的联合创始人，他以出色的讲故事能力和日本动画长篇电影的创作者而获得了国际赞誉，并被广泛认为是动画史上最有成就的电影制作人之一。\n宫崎骏出生于日本帝国的东京市，从小就对漫画和动画表现出兴趣，1963年加入了东映动画。在东映动画的早年，他担任中间画师，后来与导演高畑勋合作。',
  'score': 25.4748477935791,
  'rank': 2},
 {'content': 'Glen Keane表示宫崎骏对华特迪士尼动画工作室有着“巨大的影响”，并自从《救援小英雄》（1990年）以来一直“是我们的传统的一部分”。迪士尼复兴时期也受到了宫崎骏电影的竞争的推动。来自Pixar和Aardman Studios的艺术家们签署了一份致敬声明，称“宫崎先生，您是我们的灵感来源！”',
  'score': 24.84897232055664,
  'rank': 3}]
```

我们可以轻松地将其转换为 LangChain 检索器！在创建时，我们可以传入任何想要的 kwargs（比如 `k`）

```python
retriever = RAG.as_langchain_retriever(k=3)
```

```python
retriever.invoke("宫崎骏创立了哪家动画工作室？")
```

```python
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
prompt = ChatPromptTemplate.from_template(
    """Answer the following question based only on the provided context:
<context>
{context}
</context>
Question: {input}"""
)
llm = ChatOpenAI()
document_chain = create_stuff_documents_chain(llm, prompt)
retrieval_chain = create_retrieval_chain(retriever, document_chain)
```

```python
retrieval_chain.invoke({"input": "What animation studio did Miyazaki found?"})
```

宫崎骏创立了吉卜力工作室。