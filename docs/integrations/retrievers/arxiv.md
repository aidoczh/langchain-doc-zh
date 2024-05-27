# Arxiv

[arXiv](https://arxiv.org/) 是一个开放获取的学术论文存档，收录了来自物理学、数学、计算机科学、数量生物学、数量金融、统计学、电气工程与系统科学以及经济学等领域的 200 万篇学术论文。

这篇文档展示了如何从 `Arxiv.org` 检索科学文章，并将其转换为下游使用的文档格式。

## 安装

首先，您需要安装 `arxiv` python 包。

```python
%pip install --upgrade --quiet  arxiv
```

`ArxivRetriever` 有以下参数：

- 可选参数 `load_max_docs`：默认值为 100。用于限制下载文档的数量。下载所有 100 篇文档需要一些时间，因此在实验中使用较小的数字。目前有一个硬性限制为 300。

- 可选参数 `load_all_available_meta`：默认值为 False。默认情况下，只下载最重要的字段：`Published`（文档发布/最后更新日期）、`Title`（标题）、`Authors`（作者）、`Summary`（摘要）。如果为 True，则还会下载其他字段。

`get_relevant_documents()` 有一个参数 `query`：用于在 `Arxiv.org` 中查找文档的自由文本。

## 示例

### 运行检索器

```python
from langchain_community.retrievers import ArxivRetriever
```
```python
retriever = ArxivRetriever(load_max_docs=2)
```
```python
docs = retriever.invoke("1605.08386")
```
```python
docs[0].metadata  # 文档的元信息
```
```output
{'Published': '2016-05-26',
 'Title': 'Heat-bath random walks with Markov bases',
 'Authors': 'Caprice Stanley, Tobias Windisch',
 'Summary': 'Graphs
```python
questions = [
    "什么是带有马尔可夫基础的热浴随机行走？包括参考文献。",
]
chat_history = []
for question in questions:
    result = qa({"question": question, "chat_history": chat_history})
    chat_history.append((question, result["answer"]))
    print(f"-> **问题**: {question} \n")
    print(f"**回答**: {result['answer']} \n")
```
```output

-> **问题**: 什么是带有马尔可夫基础的热浴随机行走？包括参考文献。 

**回答**: 带有马尔可夫基础的热浴随机行走（HB-MB）是一类在统计力学和凝聚态物理领域研究的随机过程。在这些过程中，粒子通过根据粒子能量和周围能量依赖的概率分布选择邻近位置进行转移，从而在晶格中移动。

HB-MB 过程是由 Bortz, Kalos, 和 Lebowitz 在 1975 年引入的，用于模拟晶格中相互作用粒子在热平衡状态下的动力学。该方法已被用于研究各种物理现象，包括相变、临界行为和输运性质。

参考文献:

Bortz, A. B., Kalos, M. H., & Lebowitz, J. L. (1975). A new algorithm for Monte Carlo simulation of Ising spin systems. Journal of Computational Physics, 17(1), 10-18.

Binder, K., & Heermann, D. W. (2010). Monte Carlo simulation in statistical physics: an introduction. Springer Science & Business Media.

```