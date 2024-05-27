# Jira

这篇笔记介绍了如何使用 `Jira` 工具包。

`Jira` 工具包允许代理与特定的 Jira 实例进行交互，执行诸如搜索问题和创建问题等操作，该工具包封装了 atlassian-python-api 库，更多信息请参见：[https://atlassian-python-api.readthedocs.io/jira.html](https://atlassian-python-api.readthedocs.io/jira.html)

要使用此工具，您必须首先设置环境变量：

    JIRA_API_TOKEN

    JIRA_USERNAME

    JIRA_INSTANCE_URL

```python
%pip install --upgrade --quiet  atlassian-python-api
```

```python
import os
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits.jira.toolkit import JiraToolkit
from langchain_community.utilities.jira import JiraAPIWrapper
from langchain_openai import OpenAI
```

```python
os.environ["JIRA_API_TOKEN"] = "abc"
os.environ["JIRA_USERNAME"] = "123"
os.environ["JIRA_INSTANCE_URL"] = "https://jira.atlassian.com"
os.environ["OPENAI_API_KEY"] = "xyz"
```

```python
llm = OpenAI(temperature=0)
jira = JiraAPIWrapper()
toolkit = JiraToolkit.from_jira_api_wrapper(jira)
agent = initialize_agent(
    toolkit.get_tools(), llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
agent.run("make a new issue in project PW to remind me to make more fried rice")
```

```output
> 进入新的 AgentExecutor 链...
 我需要在项目 PW 中创建一个问题
操作：创建问题
操作输入：{"summary": "Make more fried rice", "description": "Reminder to make more fried rice", "issuetype": {"name": "Task"}, "priority": {"name": "Low"}, "project": {"key": "PW"}}
观察：无
思考：我现在知道最终答案
最终答案：在项目 PW 中创建了一个名为“Make more fried rice”、描述为“Reminder to make more fried rice”的新问题。
> 链结束。
```

```output
'在项目 PW 中创建了一个名为“Make more fried rice”、描述为“Reminder to make more fried rice”的新问题。'
```