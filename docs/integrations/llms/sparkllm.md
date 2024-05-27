# SparkLLM

[SparkLLM](https://xinghuo.xfyun.cn/spark) 是科大讯飞独立开发的大规模认知模型。通过学习大量文本、代码和图像，它具有跨领域的知识和语言理解能力。SparkLLM 能够基于自然对话理解并执行任务。

## 先决条件

- 从[iFlyTek SparkLLM API控制台](https://console.xfyun.cn/services/bm3)获取 SparkLLM 的 app_id、api_key 和 api_secret（更多信息请参阅[iFlyTek SparkLLM介绍](https://xinghuo.xfyun.cn/sparkapi)），然后设置环境变量 `IFLYTEK_SPARK_APP_ID`、`IFLYTEK_SPARK_API_KEY` 和 `IFLYTEK_SPARK_API_SECRET`，或者在创建 `ChatSparkLLM` 时传递参数，就像上面的演示一样。

## 使用 SparkLLM

```python
import os
os.environ["IFLYTEK_SPARK_APP_ID"] = "app_id"
os.environ["IFLYTEK_SPARK_API_KEY"] = "api_key"
os.environ["IFLYTEK_SPARK_API_SECRET"] = "api_secret"
```

```python
from langchain_community.llms import SparkLLM
# 加载模型
llm = SparkLLM()
res = llm.invoke("What's your name?")
print(res)
```

```output
/Users/liugddx/code/langchain/libs/core/langchain_core/_api/deprecation.py:117: LangChainDeprecationWarning: 函数 `__call__` 在 LangChain 0.1.7 中已弃用，并将在 0.2.0 中移除。请改用 invoke。
  warn_deprecated(
```

```output
我的名字是科大讯飞 Spark。今天我能为您做些什么？
```

```python
res = llm.generate(prompts=["hello!"])
res
```

```output
LLMResult(generations=[[Generation(text='Hello! How can I assist you today?')]], llm_output=None, run=[RunInfo(run_id=UUID('d8cdcd41-a698-4cbf-a28d-e74f9cd2037b'))])
```

```python
for res in llm.stream("foo:"):
    print(res)
```

```output
Hello! How can I assist you today?
```