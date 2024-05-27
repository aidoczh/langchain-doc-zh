# 百川LLM

百川公司（https://www.baichuan-ai.com/）是一家中国初创公司，致力于解决人类的基本需求：效率、健康和幸福。

## 先决条件

访问百川LLM API 需要一个 API 密钥。请访问 https://platform.baichuan-ai.com/ 获取您的 API 密钥。

## 使用百川LLM

```python
import os
os.environ["BAICHUAN_API_KEY"] = "YOUR_API_KEY"
```

```python
from langchain_community.llms import BaichuanLLM
# 加载模型
llm = BaichuanLLM()
res = llm.invoke("What's your name?")
print(res)
```

```python
res = llm.generate(prompts=["你好！"])
res
```

```python
for res in llm.stream("Who won the second world war?"):
    print(res)
```

```python
import asyncio
async def run_aio_stream():
    async for res in llm.astream("Write a poem about the sun."):
        print(res)
asyncio.run(run_aio_stream())
```