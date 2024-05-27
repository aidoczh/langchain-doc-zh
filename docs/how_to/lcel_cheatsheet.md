# LangChain 表达语言速查表
这是所有最重要的 LCEL 原语的快速参考。更高级的用法请参阅[LCEL 如何指南](/docs/how_to/#langchain-expression-language-lcel)和[完整的 API 参考](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html)。
### 调用一个可运行对象
#### [Runnable.invoke()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.invoke) / [Runnable.ainvoke()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.ainvoke)
```python
from langchain_core.runnables import RunnableLambda
runnable = RunnableLambda(lambda x: str(x))
runnable.invoke(5)
# 异步变体:
# await runnable.ainvoke(5)
```
```output
'5'
```
### 批处理可运行对象
#### [Runnable.batch()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.batch) / [Runnable.abatch()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.abatch)
```python
from langchain_core.runnables import RunnableLambda
runnable = RunnableLambda(lambda x: str(x))
runnable.batch([7, 8, 9])
# 异步变体:
# await runnable.abatch([7, 8, 9])
```
```output
['7', '8', '9']
```
### 流式处理可运行对象
#### [Runnable.stream()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.stream) / [Runnable.astream()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.astream)
```python
from langchain_core.runnables import RunnableLambda
def func(x):
    for y in x:
        yield str(y)
runnable = RunnableLambda(func)
for chunk in runnable.stream(range(5)):
    print(chunk)
# 异步变体:
# async for chunk in await runnable.astream(range(5)):
#     print(chunk)
```
```output
0
1
2
3
4
```
### 组合可运行对象
#### 管道运算符 `|`
```python
from langchain_core.runnables import RunnableLambda
runnable1 = RunnableLambda(lambda x: {"foo": x})
runnable2 = RunnableLambda(lambda x: [x] * 2)
chain = runnable1 | runnable2
chain.invoke(2)
```
```output
[{'foo': 2}, {'foo': 2}]
```
### 并行调用可运行对象
#### [RunnableParallel](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableParallel.html)
```python
from langchain_core.runnables import RunnableLambda, RunnableParallel
runnable1 = RunnableLambda(lambda x: {"foo": x})
runnable2 = RunnableLambda(lambda x: [x] * 2)
chain = RunnableParallel(first=runnable1, second=runnable2)
chain.invoke(2)
```
```output
{'first': {'foo': 2}, 'second': [2, 2]}
```
### 将任何函数转换为可运行对象
#### [RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html)
```python
from langchain_core.runnables import RunnableLambda
def func(x):
    return x + 5
runnable = RunnableLambda(func)
runnable.invoke(2)
```
```output
7
```
### 合并输入和输出字典
#### [RunnablePassthrough.assign](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.passthrough.RunnablePassthrough.html)
```python
from langchain_core.runnables import RunnableLambda, RunnablePassthrough
runnable1 = RunnableLambda(lambda x: x["foo"] + 7)
chain = RunnablePassthrough.assign(bar=runnable1)
chain.invoke({"foo": 10})
```
```output
{'foo': 10, 'bar': 17}
```
### 在输出字典中包含输入字典
#### [RunnablePassthrough](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.passthrough.RunnablePassthrough.html)
```python
from langchain_core.runnables import (
    RunnableLambda,
    RunnableParallel,
    RunnablePassthrough,
)
runnable1 = RunnableLambda(lambda x: x["foo"] + 7)
chain = RunnableParallel(bar=runnable1, baz=RunnablePassthrough())
chain.invoke({"foo": 10})
```
```output
{'bar': 17, 'baz': {'foo': 10}}
```
### 添加默认调用参数
#### [Runnable.bind](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.bind)
```python
from typing import Optional
from langchain_core.runnables import RunnableLambda
def func(main_arg: dict, other_arg: Optional[str] = None) -> dict:
    if other_arg:
        return {**main_arg, **{"foo": other_arg}}
    return main_arg
runnable1 = RunnableLambda(func)
bound_runnable1 = runnable1.bind(other_arg="bye")
bound_runnable1.invoke({"bar": "hello"})
```python
chain.invoke(7, config={"configurable": {"second_step": "string"}})
```
```output
'7'
```
```python
chain.invoke(7, config={"configurable": {"second_step": "list"}})
```
```output
[{'foo': 7}]
```
```python
chain.invoke(7)
```
```output
{'foo': 7}
```
### 根据输入动态构建链
```python
chain.invoke(7, config={"configurable": {"second_step": "string"}})
```
```output
"{'foo': 7}"
```
```python
chain.invoke(7)
```
```output
[{'foo': 7}]
```
```python
chain.invoke(5)
```
```output
[5, 5]
```
从输入动态构建链，根据输入值的大小选择不同的运行方式。当输入为7时，输出为`{'foo': 7}`；当输入为5时，输出为`[5, 5]`。
### 生成事件流
#### [Runnable.astream_events](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.astream_events)
```python
# | echo: false
import nest_asyncio
nest_asyncio.apply()
```
```python
from langchain_core.runnables import RunnableLambda, RunnableParallel
runnable1 = RunnableLambda(lambda x: {"foo": x}, name="first")
async def func(x):
    for _ in range(5):
        yield x
runnable2 = RunnableLambda(func, name="second")
chain = runnable1 | runnable2
async for event in chain.astream_events("bar", version="v2"):
    print(f"event={event['event']} | name={event['name']} | data={event['data']}")
```
```output
event=on_chain_start | name=RunnableSequence | data={'input': 'bar'}
event=on_chain_start | name=first | data={}
event=on_chain_stream | name=first | data={'chunk': {'foo': 'bar'}}
event=on_chain_start | name=second | data={}
event=on_chain_end | name=first | data={'output': {'foo': 'bar'}, 'input': 'bar'}
event=on_chain_stream | name=second | data={'chunk': {'foo': 'bar'}}
event=on_chain_stream | name=RunnableSequence | data={'chunk': {'foo': 'bar'}}
event=on_chain_stream | name=second | data={'chunk': {'foo': 'bar'}}
event=on_chain_stream | name=RunnableSequence | data={'chunk': {'foo': 'bar'}}
event=on_chain_stream | name=second | data={'chunk': {'foo': 'bar'}}
event=on_chain_stream | name=RunnableSequence | data={'chunk': {'foo': 'bar'}}
event=on_chain_stream | name=second | data={'chunk': {'foo': 'bar'}}
event=on_chain_stream | name=RunnableSequence | data={'chunk': {'foo': 'bar'}}
event=on_chain_end | name=second | data={'output': {'foo': 'bar'}, 'input': {'foo': 'bar'}}
event=on_chain_end | name=RunnableSequence | data={'output': {'foo': 'bar'}}
```
### 在完成时返回批量输出
#### [Runnable.batch_as_completed](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.batch_as_completed) / [Runnable.abatch_as_completed](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.abatch_as_completed)
```python
import time
from langchain_core.runnables import RunnableLambda, RunnableParallel
runnable1 = RunnableLambda(lambda x: time.sleep(x) or print(f"slept {x}"))
for idx, result in runnable1.batch_as_completed([5, 1]):
    print(idx, result)
```
```output
slept 1
1 None
slept 5
0 None
```
### 返回输出字典的子集
#### [Runnable.pick](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.pick)
```python
from langchain_core.runnables import RunnableLambda, RunnablePassthrough
runnable1 = RunnableLambda(lambda x: x["baz"] + 5)
chain = RunnablePassthrough.assign(foo=runnable1).pick(["foo", "bar"])
chain.invoke({"bar": "hi", "baz": 2})
```
```output
{'foo': 7, 'bar': 'hi'}
```
### 声明式地创建可批量运行的版本
#### [Runnable.map](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.map)
```python
from langchain_core.runnables import RunnableLambda
runnable1 = RunnableLambda(lambda x: list(range(x)))
runnable2 = RunnableLambda(lambda x: x + 5)
chain = runnable1 | runnable2.map()
chain.invoke(3)
```
```output
[5, 6, 7]
```
### 获取可运行对象的图形表示
#### [Runnable.get_graph](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.get_graph)
```python
from langchain_core.runnables import RunnableLambda, RunnableParallel
runnable1 = RunnableLambda(lambda x: {"foo": x})
runnable2 = RunnableLambda(lambda x: [x] * 2)
runnable3 = RunnableLambda(lambda x: str(x))
```
链 = runnable1 | RunnableParallel(second=runnable2, third=runnable3)
chain.get_graph().print_ascii()
```
```output
                             +-------------+                              
| LambdaInput |
                             +-------------+                              
                                    *                                     
                                    *                                     
                                    *                                     
                    +------------------------------+                      
| Lambda(lambda x: {'foo': x}) |
                    +------------------------------+                      
                                    *                                     
                                    *                                     
                                    *                                     
                     +-----------------------------+                      
| Parallel<second,third>Input |
                     +-----------------------------+                      
                        ****                  ***                         
                    ****                         ****                     
                  **                                 **                   
+---------------------------+               +--------------------------+  
| Lambda(lambda x: [x] * 2) |               | Lambda(lambda x: str(x)) |
+---------------------------+               +--------------------------+  
                        ****                  ***                         
                            ****          ****                            
                                **      **                                
                    +------------------------------+                      
| Parallel<second,third>Output |
                    +------------------------------+
```
### 获取链中的所有提示
#### [Runnable.get_prompts](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.get_prompts)
```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableLambda
prompt1 = ChatPromptTemplate.from_messages(
    [("system", "good ai"), ("human", "{input}")]
)
prompt2 = ChatPromptTemplate.from_messages(
    [
        ("system", "really good ai"),
        ("human", "{input}"),
        ("ai", "{ai_output}"),
        ("human", "{input2}"),
    ]
)
fake_llm = RunnableLambda(lambda prompt: "i am good ai")
chain = prompt1.assign(ai_output=fake_llm) | prompt2 | fake_llm
for i, prompt in enumerate(chain.get_prompts()):
    print(f"**prompt {i=}**\n")
    print(prompt.pretty_repr())
    print("\n" * 3)
```
```output
**prompt i=0**
================================ 系统消息 ================================
good ai
================================ 用户消息 =================================
{input}
**prompt i=1**
================================ 系统消息 ================================
really good ai
================================ 用户消息 =================================
{input}
================================== AI 消息 ==================================
{ai_output}
================================ 用户消息 =================================
{input2}
```
### 添加生命周期监听器
#### [Runnable.with_listeners](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.with_listeners)
```python
import time
from langchain_core.runnables import RunnableLambda
from langchain_core.tracers.schemas import Run
def on_start(run_obj: Run):
    print("start_time:", run_obj.start_time)
def on_end(run_obj: Run):
    print("end_time:", run_obj.end_time)
runnable1 = RunnableLambda(lambda x: time.sleep(x))
chain = runnable1.with_listeners(on_start=on_start, on_end=on_end)
chain.invoke(2)
```
```output
start_time: 2024-05-17 23:04:00.951065+00:00
end_time: 2024-05-17 23:04:02.958765+00:00
```