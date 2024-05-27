# NetworkX

[NetworkX](https://networkx.org/) 是一个用于创建、操作和研究复杂网络结构、动态和功能的 Python 包。

本笔记将介绍如何在图数据结构上进行问答。

## 设置

我们需要安装一个 Python 包。

```python
%pip install --upgrade --quiet  networkx
```

## 创建图

在这一部分，我们构建一个示例图。目前，这对于小段文本效果最佳。

```python
from langchain.indexes import GraphIndexCreator
from langchain_openai import OpenAI
```

```python
index_creator = GraphIndexCreator(llm=OpenAI(temperature=0))
```

```python
with open("../../../how_to/state_of_the_union.txt") as f:
    all_text = f.read()
```

我们将只使用一个小片段，因为目前提取知识三元组有点密集。

```python
text = "\n".join(all_text.split("\n\n")[105:108])
```

```python
text
```

```output
'It won’t look like much, but if you stop and look closely, you’ll see a “Field of dreams,” the ground on which America’s future will be built. \nThis is where Intel, the American company that helped build Silicon Valley, is going to build its $20 billion semiconductor “mega site”. \nUp to eight state-of-the-art factories in one place. 10,000 new good-paying jobs. '
```

```python
graph = index_creator.from_text(text)
```

我们可以检查创建的图。

```python
graph.get_triples()
```

```output
[('Intel', '$20 billion semiconductor "mega site"', 'is going to build'),
 ('Intel', 'state-of-the-art factories', 'is building'),
 ('Intel', '10,000 new good-paying jobs', 'is creating'),
 ('Intel', 'Silicon Valley', 'is helping build'),
 ('Field of dreams',
  "America's future will be built",
  'is the ground on which')]
```

## 查询图

现在我们可以使用图问答链来询问图的问题。

```python
from langchain.chains import GraphQAChain
```

```python
chain = GraphQAChain.from_llm(OpenAI(temperature=0), graph=graph, verbose=True)
```

```python
chain.run("what is Intel going to build?")
```

```output
' Intel is going to build a $20 billion semiconductor "mega site" with state-of-the-art factories, creating 10,000 new good-paying jobs and helping to build Silicon Valley.'
```

## 保存图

我们还可以保存和加载图。

```python
graph.write_to_gml("graph.gml")
```

```python
from langchain.indexes.graph import NetworkxEntityGraph
```

```python
loaded_graph = NetworkxEntityGraph.from_gml("graph.gml")
```

```python
loaded_graph.get_triples()
```

```output
[('Intel', '$20 billion semiconductor "mega site"', 'is going to build'),
 ('Intel', 'state-of-the-art factories', 'is building'),
 ('Intel', '10,000 new good-paying jobs', 'is creating'),
 ('Intel', 'Silicon Valley', 'is helping build'),
 ('Field of dreams',
  "America's future will be built",
  'is the ground on which')]
```

```python
loaded_graph.get_number_of_nodes()
```

```python
loaded_graph.add_node("NewNode")
```

```python
loaded_graph.has_node("NewNode")
```

```python
loaded_graph.remove_node("NewNode")
```

```python
loaded_graph.get_neighbors("Intel")
```

```python
loaded_graph.has_edge("Intel", "Silicon Valley")
```

```python
loaded_graph.remove_edge("Intel", "Silicon Valley")
```

```python
loaded_graph.clear_edges()
```

```python
loaded_graph.clear()
```