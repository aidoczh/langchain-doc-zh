# RELLM

[RELLM](https://github.com/r2d4/rellm) 是一个库，用于对结构化解码的本地 Hugging Face pipeline 模型进行封装。

它通过逐个生成标记来工作。在每一步中，它会屏蔽不符合提供的部分正则表达式的标记。

**警告 - 这个模块仍处于实验阶段**

```python
%pip install --upgrade --quiet  rellm langchain-huggingface > /dev/null
```

### Hugging Face 基准线

首先，让我们通过检查模型在没有结构化解码的情况下的输出来建立一个定性基准线。

```python
import logging
logging.basicConfig(level=logging.ERROR)
prompt = """Human: "What's the capital of the United States?"
AI Assistant:{
  "action": "Final Answer",
  "action_input": "The capital of the United States is Washington D.C."
}
Human: "What's the capital of Pennsylvania?"
AI Assistant:{
  "action": "Final Answer",
  "action_input": "The capital of Pennsylvania is Harrisburg."
}
Human: "What 2 + 5?"
AI Assistant:{
  "action": "Final Answer",
  "action_input": "2 + 5 = 7."
}
Human: 'What's the capital of Maryland?'
AI Assistant:"""
```

```python
from langchain_huggingface import HuggingFacePipeline
from transformers import pipeline
hf_model = pipeline(
    "text-generation", model="cerebras/Cerebras-GPT-590M", max_new_tokens=200
)
original_model = HuggingFacePipeline(pipeline=hf_model)
generated = original_model.generate([prompt], stop=["Human:"])
print(generated)
```

```output
Setting `pad_token_id` to `eos_token_id`:50256 for open-end generation.
``````output
generations=[[Generation(text=' "What\'s the capital of Maryland?"\n', generation_info=None)]] llm_output=None
```

***这并不那么令人印象深刻，是吗？它没有回答问题，也没有遵循 JSON 格式！让我们尝试使用结构化解码。***

## RELLM LLM 封装器

让我们再试一次，这次提供一个正则表达式来匹配 JSON 结构化格式。

```python
import regex  # 请注意，这是正则表达式库，而不是 Python 的 re 标准库模块
# 我们将选择一个与结构化 JSON 字符串匹配的正则表达式，看起来像:
# {
#  "action": "Final Answer",
# "action_input": string or dict
# }
pattern = regex.compile(
    r'\{\s*"action":\s*"Final Answer",\s*"action_input":\s*(\{.*\}|"[^"]*")\s*\}\nHuman:'
)
```

```python
from langchain_experimental.llms import RELLM
model = RELLM(pipeline=hf_model, regex=pattern, max_new_tokens=200)
generated = model.predict(prompt, stop=["Human:"])
print(generated)
```

```output
{"action": "Final Answer",
  "action_input": "The capital of Maryland is Baltimore."
}
```

**哇！没有解析错误。**