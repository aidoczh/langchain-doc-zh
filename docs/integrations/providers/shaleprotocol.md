# Shale Protocol

[Shale Protocol](https://shaleprotocol.com) 提供了针对开放式大型语言模型（LLMs）的生产就绪推理 API。它是一个即插即用的 API，托管在高度可扩展的 GPU 云基础设施上。

我们的免费套餐支持每个密钥每日最多 1K 次请求，因为我们希望消除任何人开始使用 LLM 构建 genAI 应用的障碍。

有了 Shale Protocol，开发人员/研究人员可以免费创建应用程序并探索开放式 LLMs 的能力。

本页面介绍了如何将 Shale-Serve API 与 LangChain 结合使用。

截至 2023 年 6 月，该 API 默认支持 Vicuna-13B。我们将在未来的版本中支持更多的 LLMs，例如 Falcon-40B。

## 如何操作

### 1. 在 https://shaleprotocol.com 上找到我们 Discord 的链接。通过我们 Discord 上的 "Shale Bot" 生成 API 密钥。无需信用卡，也没有免费试用。这是一个永久免费套餐，每个 API 密钥每天限制 1K 次。

### 2. 使用 https://shale.live/v1 作为 OpenAI API 的替代

例如

```python
from langchain_openai import OpenAI
from langchain_core.prompts import PromptTemplate
from langchain.chains import LLMChain
import os
os.environ['OPENAI_API_BASE'] = "https://shale.live/v1"
os.environ['OPENAI_API_KEY'] = "输入你的 API 密钥"
llm = OpenAI()
template = """问题：{question}
# 答案：让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.run(question)
```