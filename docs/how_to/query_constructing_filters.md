---

sidebar_position: 6

---

# 如何构建用于查询分析的过滤器

我们可能希望进行查询分析，以提取过滤器以传递给检索器。我们要求 LLM 代表这些过滤器的一种方式是作为 Pydantic 模型。然后就涉及将该 Pydantic 模型转换为可传递给检索器的过滤器的问题。

这可以通过手动完成，但 LangChain 还提供了一些“转换器”，能够将通用语法转换为每个检索器特定的过滤器。在这里，我们将介绍如何使用这些转换器。

```python
from typing import Optional
from langchain.chains.query_constructor.ir import (
    Comparator,
    Comparison,
    Operation,
    Operator,
    StructuredQuery,
)
from langchain.retrievers.self_query.chroma import ChromaTranslator
from langchain.retrievers.self_query.elasticsearch import ElasticsearchTranslator
from langchain_core.pydantic_v1 import BaseModel
```

在这个例子中，`year` 和 `author` 都是要进行过滤的属性。

```python
class Search(BaseModel):
    query: str
    start_year: Optional[int]
    author: Optional[str]
```

```python
search_query = Search(query="RAG", start_year=2022, author="LangChain")
```

```python
def construct_comparisons(query: Search):
    comparisons = []
    if query.start_year is not None:
        comparisons.append(
            Comparison(
                comparator=Comparator.GT,
                attribute="start_year",
                value=query.start_year,
            )
        )
    if query.author is not None:
        comparisons.append(
            Comparison(
                comparator=Comparator.EQ,
                attribute="author",
                value=query.author,
            )
        )
    return comparisons
```

```python
comparisons = construct_comparisons(search_query)
```

```python
_filter = Operation(operator=Operator.AND, arguments=comparisons)
```

```python
ElasticsearchTranslator().visit_operation(_filter)
```

```output
{'bool': {'must': [{'range': {'metadata.start_year': {'gt': 2022}}},
   {'term': {'metadata.author.keyword': 'LangChain'}}]}}
```

```python
ChromaTranslator().visit_operation(_filter)
```

```output
{'$and': [{'start_year': {'$gt': 2022}}, {'author': {'$eq': 'LangChain'}}]}
```