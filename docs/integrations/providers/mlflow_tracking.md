# MLflow

[MLflow](https://www.mlflow.org/docs/latest/what-is-mlflow) 是一个多功能、可扩展的开源平台，用于管理机器学习生命周期中的工作流程和工件。它内置了许多流行的 ML 库集成，但也可以与任何库、算法或部署工具一起使用。它被设计为可扩展的，因此您可以编写插件来支持新的工作流程、库和工具。

这个笔记本介绍了如何将您的 LangChain 实验跟踪到您的 `MLflow 服务器` 中。

## 外部示例

`MLflow` 提供了 [几个示例](https://github.com/mlflow/mlflow/tree/master/examples/langchain) 来展示与 `LangChain` 集成：

- [simple_chain](https://github.com/mlflow/mlflow/blob/master/examples/langchain/simple_chain.py)

- [simple_agent](https://github.com/mlflow/mlflow/blob/master/examples/langchain/simple_agent.py)

- [retriever_chain](https://github.com/mlflow/mlflow/blob/master/examples/langchain/retriever_chain.py)

- [retrieval_qa_chain](https://github.com/mlflow/mlflow/blob/master/examples/langchain/retrieval_qa_chain.py)

## 示例

```python
%pip install --upgrade --quiet  azureml-mlflow
%pip install --upgrade --quiet  pandas
%pip install --upgrade --quiet  textstat
%pip install --upgrade --quiet  spacy
%pip install --upgrade --quiet  langchain-openai
%pip install --upgrade --quiet  google-search-results
!python -m spacy download en_core_web_sm
```

```python
import os
os.environ["MLFLOW_TRACKING_URI"] = ""
os.environ["OPENAI_API_KEY"] = ""
os.environ["SERPAPI_API_KEY"] = ""
```

```python
from langchain_community.callbacks import MlflowCallbackHandler
from langchain_openai import OpenAI
```

```python
"""主函数。
此函数用于尝试回调处理程序。
场景：
1. OpenAI LLM
2. 多代际多子链的链
3. 带工具的代理
"""
mlflow_callback = MlflowCallbackHandler()
llm = OpenAI(
    model_name="gpt-3.5-turbo", temperature=0, callbacks=[mlflow_callback], verbose=True
)
```

```python
# 场景 1 - LLM
llm_result = llm.generate(["Tell me a joke"])
mlflow_callback.flush_tracker(llm)
```

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
```

```python
# 场景 2 - 链
template = """You are a playwright. Given the title of play, it is your job to write a synopsis for that title.
Title: {title}
Playwright: This is a synopsis for the above play:"""
prompt_template = PromptTemplate(input_variables=["title"], template=template)
synopsis_chain = LLMChain(llm=llm, prompt=prompt_template, callbacks=[mlflow_callback])
test_prompts = [
    {
        "title": "documentary about good video games that push the boundary of game design"
    },
]
synopsis_chain.apply(test_prompts)
mlflow_callback.flush_tracker(synopsis_chain)
```

```python
from langchain.agents import AgentType, initialize_agent, load_tools
```

```python
# 场景 3 - 带工具的代理
tools = load_tools(["serpapi", "llm-math"], llm=llm, callbacks=[mlflow_callback])
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    callbacks=[mlflow_callback],
    verbose=True,
)
agent.run(
    "Who is Leo DiCaprio's girlfriend? What is her current age raised to the 0.43 power?"
)
mlflow_callback.flush_tracker(agent, finish=True)
```