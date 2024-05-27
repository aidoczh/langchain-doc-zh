# 香蕉

[香蕉](https://www.banana.dev/about-us) 专注于构建机器学习基础设施。

这个示例介绍了如何使用 LangChain 与香蕉模型进行交互。

```python
# 安装包 https://docs.banana.dev/banana-docs/core-concepts/sdks/python
%pip install --upgrade --quiet  banana-dev
```

```python
# 获取新的令牌：https://app.banana.dev/
# 我们需要三个参数来调用 Banana.dev API：
# * 团队 API 密钥
# * 模型的唯一密钥
# * 模型的 URL 别名
import os
# 您可以从 https://app.banana.dev 的主仪表板获取此信息
os.environ["BANANA_API_KEY"] = "YOUR_API_KEY"
# 或者
# BANANA_API_KEY = getpass()
```

```python
from langchain.chains import LLMChain
from langchain_community.llms import Banana
from langchain_core.prompts import PromptTemplate
```

```python
template = """问题: {question}
答案: 让我们逐步思考。"""
prompt = PromptTemplate.from_template(template)
```

```python
# 这两个参数都可以在您模型的详细页面 https://app.banana.dev 找到
llm = Banana(model_key="YOUR_MODEL_KEY", model_url_slug="YOUR_MODEL_URL_SLUG")
```

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

```python
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.run(question)
```