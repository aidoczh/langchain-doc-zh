# GPT4All

[GPT4All](https://github.com/nomic-ai/gpt4all) 是一个开源聊天机器人生态系统，它经过大量干净的助手数据（包括代码、故事和对话）训练而成。

以下是如何使用 LangChain 与 `GPT4All` 模型进行交互的示例。

```python
%pip install --upgrade --quiet  gpt4all > /dev/null
```

```output
Note: you may need to restart the kernel to use updated packages.
```

### 导入 GPT4All

```python
from langchain.chains import LLMChain
from langchain_community.llms import GPT4All
from langchain_core.callbacks import StreamingStdOutCallbackHandler
from langchain_core.prompts import PromptTemplate
```

### 设置问题传递给 LLM

```python
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
```

### 指定模型

要在本地运行，需下载兼容的 ggml 格式模型。

[gpt4all 页面](https://gpt4all.io/index.html) 有一个有用的 `Model Explorer` 部分：

* 选择感兴趣的模型

* 使用 UI 下载并将 `.bin` 文件移动到 `local_path`（如下所示）

欲了解更多信息，请访问 https://github.com/nomic-ai/gpt4all。

---

```python
local_path = (
    "./models/ggml-gpt4all-l13b-snoozy.bin"  # 替换为您所需的本地文件路径
)
```

```python
# 回调支持逐标记流式处理
callbacks = [StreamingStdOutCallbackHandler()]
# 需要 verbose 参数传递给回调管理器
llm = GPT4All(model=local_path, callbacks=callbacks, verbose=True)
# 如果要使用自定义模型，请添加 backend 参数
# 请查看 https://docs.gpt4all.io/gpt4all_python.html 了解支持的后端
llm = GPT4All(model=local_path, backend="gptj", callbacks=callbacks, verbose=True)
```

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

```python
question = "What NFL team won the Super Bowl in the year Justin Bieber was born?"
llm_chain.run(question)
```

贾斯汀·比伯出生于1994年3月1日。1994年，牛仔队赢得了第二十八届超级碗。