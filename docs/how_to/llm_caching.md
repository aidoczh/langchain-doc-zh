# 如何缓存LLM响应

LangChain为LLM提供了一个可选的缓存层。这有两个好处：

- 如果您经常多次请求相同的完成结果，缓存可以通过减少对LLM提供商的API调用来节省费用。

- 通过减少对LLM提供商的API调用次数，可以加快应用程序的速度。

```python
from langchain.globals import set_llm_cache
from langchain_openai import OpenAI
# 为了让缓存效果更明显，让我们使用一个速度较慢的模型。
llm = OpenAI(model_name="gpt-3.5-turbo-instruct", n=2, best_of=2)
```

```python
%%time
from langchain.cache import InMemoryCache
set_llm_cache(InMemoryCache())
# 第一次，因为尚未缓存，所以需要更长的时间
llm.predict("Tell me a joke")
```

```output
CPU times: user 13.7 ms, sys: 6.54 ms, total: 20.2 ms
Wall time: 330 ms
```

```output
"\n\n为什么自行车不能自己站起来？因为它太累了！"
```

```python
%%time
# 第二次则会更快
llm.predict("Tell me a joke")
```

```output
CPU times: user 436 µs, sys: 921 µs, total: 1.36 ms
Wall time: 1.36 ms
```

```output
"\n\n为什么自行车不能自己站起来？因为它太累了！"
```

## SQLite缓存

```python
!rm .langchain.db
```

```python
# 我们也可以使用SQLite缓存来实现相同的效果
from langchain_community.cache import SQLiteCache
set_llm_cache(SQLiteCache(database_path=".langchain.db"))
```

```python
%%time
# 第一次，因为尚未缓存，所以需要更长的时间
llm.predict("Tell me a joke")
```

```output
CPU times: user 29.3 ms, sys: 17.3 ms, total: 46.7 ms
Wall time: 364 ms
```

```output
'\n\n番茄为什么变红了？\n\n因为它看到了沙拉酱！'
```

```python
%%time
# 第二次则会更快
llm.predict("Tell me a joke")
```

```output
CPU times: user 4.58 ms, sys: 2.23 ms, total: 6.8 ms
Wall time: 4.68 ms
```

```output
'\n\n番茄为什么变红了？\n\n因为它看到了沙拉酱！'
```