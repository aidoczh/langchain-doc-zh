# 预测守卫

```python
%pip install --upgrade --quiet predictionguard langchain
```

```python
import os
from langchain.chains import LLMChain
from langchain_community.llms import PredictionGuard
from langchain_core.prompts import PromptTemplate
```

## 基本 LLM 使用

```python
# 可选，添加你的 OpenAI API 密钥。这是可选的，因为 Prediction Guard 允许你访问所有最新的开放访问模型（请参阅 https://docs.predictionguard.com）
os.environ["OPENAI_API_KEY"] = "<你的 OpenAI API 密钥>"
# 你的 Prediction Guard API 密钥。在 predictionguard.com 获取
os.environ["PREDICTIONGUARD_TOKEN"] = "<你的 Prediction Guard 访问令牌>"
```

```python
pgllm = PredictionGuard(model="OpenAI-text-davinci-003")
```

```python
pgllm("告诉我一个笑话")
```

## 控制 LLM 的输出结构/类型

```python
template = """根据上下文回答以下问题。
上下文：每条评论、私信和电子邮件建议都引领我们做出这个令人兴奋的公告！🎉 我们正式添加了两个新的蜡烛订阅盒选项！📦
独家蜡烛盒 - $80
月度蜡烛盒 - $45（全新！）
本月香味盒 - $28（全新！）
前往故事了解每个盒子的所有详细信息！👆 奖励：使用代码 50OFF 可以节省首个盒子的 50%！🎉
问题：{query}
结果："""
prompt = PromptTemplate.from_template(template)
```

```python
# 没有对 LLM 的输出进行“保护”或控制。
pgllm(prompt.format(query="这是什么样的帖子？"))
```

```python
# 使用“保护”或控制 LLM 的输出。请参阅 Prediction Guard 文档（https://docs.predictionguard.com）以了解如何使用整数、浮点数、布尔值、JSON 和其他类型和结构来控制输出。
pgllm = PredictionGuard(
    model="OpenAI-text-davinci-003",
    output={
        "type": "categorical",
        "categories": ["产品公告", "道歉", "关系型"],
    },
)
pgllm(prompt.format(query="这是什么样的帖子？"))
```

## 链接

```python
pgllm = PredictionGuard(model="OpenAI-text-davinci-003")
```

```python
template = """问题：{question}
回答：让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
llm_chain = LLMChain(prompt=prompt, llm=pgllm, verbose=True)
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.predict(question=question)
```

```python
template = """写一首关于 {subject} 的 {adjective} 诗。"""
prompt = PromptTemplate.from_template(template)
llm_chain = LLMChain(prompt=prompt, llm=pgllm, verbose=True)
llm_chain.predict(adjective="悲伤的", subject="鸭子")
```