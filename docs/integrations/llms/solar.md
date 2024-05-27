# 太阳能

*此社区集成已被弃用。您应该使用[`ChatUpstage`](../../chat/upstage)来通过聊天模型连接器访问太阳能LLM。*

```python
import os
from langchain_community.llms.solar import Solar
os.environ["SOLAR_API_KEY"] = "SOLAR_API_KEY"
llm = Solar()
llm.invoke("tell me a story?")
```

```python
from langchain.chains import LLMChain
from langchain_community.llms.solar import Solar
from langchain_core.prompts import PromptTemplate
template = """问题: {question}
回答: 让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
llm = Solar()
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支NFL球队？"
llm_chain.run(question)
```