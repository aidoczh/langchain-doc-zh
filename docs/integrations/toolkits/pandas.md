# Pandas 数据框

本笔记展示了如何使用代理与 `Pandas 数据框` 进行交互。它主要针对问题回答进行了优化。

**注意：此代理在幕后调用 `Python` 代理，该代理执行由 LLM 生成的 Python 代码 - 如果 LLM 生成的 Python 代码有害，这可能是不好的。请谨慎使用。**

```python
from langchain.agents.agent_types import AgentType
from langchain_experimental.agents.agent_toolkits import create_pandas_dataframe_agent
from langchain_openai import ChatOpenAI
```

```python
import pandas as pd
from langchain_openai import OpenAI
df = pd.read_csv(
    "https://raw.githubusercontent.com/pandas-dev/pandas/main/doc/data/titanic.csv"
)
```

## 使用 `ZERO_SHOT_REACT_DESCRIPTION`

这展示了如何使用 `ZERO_SHOT_REACT_DESCRIPTION` 代理类型初始化代理。

```python
agent = create_pandas_dataframe_agent(OpenAI(temperature=0), df, verbose=True)
```

## 使用 OpenAI 函数

这展示了如何使用 `OPENAI_FUNCTIONS` 代理类型初始化代理。请注意，这是上述方法的另一种选择。

```python
agent = create_pandas_dataframe_agent(
    ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613"),
    df,
    verbose=True,
    agent_type=AgentType.OPENAI_FUNCTIONS,
)
```

```python
agent.invoke("有多少行？")
```

```output
'There are 891 rows in the dataframe.'
```

```python
agent.invoke("有多少人有超过 3 个兄弟姐妹？")
```

```output
'30 people have more than 3 siblings.'
```

```python
agent.invoke("平均年龄的平方根是多少？")
```

```output
'The square root of the average age is 5.449689683556195.'
```

## 多数据框示例

接下来展示了代理如何与作为列表传入的多个数据框进行交互。

```python
df1 = df.copy()
df1["Age"] = df1["Age"].fillna(df1["Age"].mean())
```

```python
agent = create_pandas_dataframe_agent(OpenAI(temperature=0), [df, df1], verbose=True)
agent.invoke("年龄列中有多少行不同？")
```

```output
'177 rows in the age column are different.'
```