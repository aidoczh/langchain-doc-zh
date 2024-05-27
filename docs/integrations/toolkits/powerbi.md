# PowerBI 数据集

本笔记本展示了一个与 `Power BI 数据集` 交互的代理程序。该代理程序回答关于数据集的更一般性问题，并且能够从错误中恢复。

需要注意的是，由于该代理程序正在积极开发中，所有答案可能不正确。它运行在 [executequery endpoint](https://learn.microsoft.com/en-us/rest/api/power-bi/datasets/execute-queries) 上，该端点不允许删除操作。

### 注意事项:

- 它依赖于使用 azure.identity 包进行身份验证，可以通过 `pip install azure-identity` 进行安装。或者您也可以使用令牌作为字符串创建 powerbi 数据集，而无需提供凭据。

- 您还可以提供一个用户名来模拟对启用了 RLS 的数据集的使用。

- 该工具包使用 LLM 来从问题中创建查询，代理程序使用 LLM 进行整体执行。

- 测试主要使用了 `gpt-3.5-turbo-instruct` 模型，codex 模型似乎表现不佳。

## 初始化

```python
from azure.identity import DefaultAzureCredential
from langchain_community.agent_toolkits import PowerBIToolkit, create_pbi_agent
from langchain_community.utilities.powerbi import PowerBIDataset
from langchain_openai import ChatOpenAI
```

```python
fast_llm = ChatOpenAI(
    temperature=0.5, max_tokens=1000, model_name="gpt-3.5-turbo", verbose=True
)
smart_llm = ChatOpenAI(temperature=0, max_tokens=100, model_name="gpt-4", verbose=True)
toolkit = PowerBIToolkit(
    powerbi=PowerBIDataset(
        dataset_id="<dataset_id>",
        table_names=["table1", "table2"],
        credential=DefaultAzureCredential(),
    ),
    llm=smart_llm,
)
agent_executor = create_pbi_agent(
    llm=fast_llm,
    toolkit=toolkit,
    verbose=True,
)
```

## 示例: 描述一个表

```python
agent_executor.run("描述 table1")
```

## 示例: 在表上进行简单查询

在这个示例中，代理程序实际上找出了获取表中行数的正确查询。

```python
agent_executor.run("table1 中有多少条记录？")
```

## 示例: 运行查询

```python
agent_executor.run("table2 中按 dimension1 有多少条记录？")
```

```python
agent_executor.run("table2 中 dimensions2 的唯一值是什么？")
```

## 示例: 添加您自己的 few-shot 提示

```python
# 虚构的例子
few_shots = """
Question: 表 revenue 中有多少行？
DAX: EVALUATE ROW("行数", COUNTROWS(revenue_details))
----
Question: 表 revenue 中年份不为空的行有多少？
DAX: EVALUATE ROW("行数", COUNTROWS(FILTER(revenue_details, revenue_details[year] <> "")))
----
Question: 表 revenue 中 value 的平均值是多少美元？
DAX: EVALUATE ROW("平均值", AVERAGE(revenue_details[dollar_value]))
----
"""
toolkit = PowerBIToolkit(
    powerbi=PowerBIDataset(
        dataset_id="<dataset_id>",
        table_names=["table1", "table2"],
        credential=DefaultAzureCredential(),
    ),
    llm=smart_llm,
    examples=few_shots,
)
agent_executor = create_pbi_agent(
    llm=fast_llm,
    toolkit=toolkit,
    verbose=True,
)
```

```python
agent_executor.run("2022 年 revenue 中 value 的最大值是多少美元？")
```