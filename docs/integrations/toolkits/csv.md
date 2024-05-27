# CSV

这篇笔记展示了如何使用代理与`CSV`格式的数据进行交互。它主要针对问答进行了优化。

**注意：这个代理在幕后调用了Pandas DataFrame代理，后者又调用了Python代理，执行LLM生成的Python代码 - 如果LLM生成的Python代码有害，这可能是不好的。请谨慎使用。**

```python
from langchain.agents.agent_types import AgentType
from langchain_experimental.agents.agent_toolkits import create_csv_agent
from langchain_openai import ChatOpenAI, OpenAI
```

## 使用`ZERO_SHOT_REACT_DESCRIPTION`

这展示了如何使用`ZERO_SHOT_REACT_DESCRIPTION`代理类型来初始化代理。

```python
agent = create_csv_agent(
    OpenAI(temperature=0),
    "titanic.csv",
    verbose=True,
    agent_type=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
)
```

## 使用OpenAI函数

这展示了如何使用`OPENAI_FUNCTIONS`代理类型来初始化代理。请注意，这是上述方法的另一种选择。

```python
agent = create_csv_agent(
    ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613"),
    "titanic.csv",
    verbose=True,
    agent_type=AgentType.OPENAI_FUNCTIONS,
)
```

```python
agent.run("有多少行？")
```

```output
Error in on_chain_start callback: 'name'
``````output
调用：`python_repl_ast`，使用 `df.shape[0]`
891数据框中有891行。
> 链结束。
```

```output
'数据框中有891行。'
```

```python
agent.run("有多少人有超过3个兄弟姐妹？")
```

```output
Error in on_chain_start callback: 'name'
``````output
调用：`python_repl_ast`，使用 `df[df['SibSp'] > 3]['PassengerId'].count()`
30数据框中有30个人有超过3个兄弟姐妹。
> 链结束。
```

```output
'数据框中有30个人有超过3个兄弟姐妹。'
```

```python
agent.run("平均年龄的平方根是多少？")
```

```output
Error in on_chain_start callback: 'name'
``````output
调用：`python_repl_ast`，使用 `import pandas as pd
import math
# 创建数据框
data = {'Age': [22, 38, 26, 35, 35]}
df = pd.DataFrame(data)
# 计算平均年龄
average_age = df['Age'].mean()
# 计算平均年龄的平方根
square_root = math.sqrt(average_age)
square_root`
5.585696017507576平均年龄的平方根约为5.59。
> 链结束。
```

```output
'平均年龄的平方根约为5.59。'
```

### 多个CSV示例

接下来展示了代理如何与作为列表传入的多个CSV文件进行交互。

```python
agent = create_csv_agent(
    ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613"),
    ["titanic.csv", "titanic_age_fillna.csv"],
    verbose=True,
    agent_type=AgentType.OPENAI_FUNCTIONS,
)
agent.run("两个数据框的年龄列有多少行是不同的？")
```

```output
Error in on_chain_start callback: 'name'
``````output
调用：`python_repl_ast`，使用 `df1['Age'].nunique() - df2['Age'].nunique()`
-1年龄列中有1行在两个数据框之间是不同的。
> 链结束。
```

```output
'年龄列中有1行在两个数据框之间是不同的。'
```