# ClickUp

[ClickUp](https://clickup.com/) 是一个全能的生产力平台，为各行各业的小型和大型团队提供灵活和可定制的工作管理解决方案、工具和功能。

它是一种面向各种规模企业的基于云的项目管理解决方案，提供通信和协作工具，帮助实现组织目标。

```python
%reload_ext autoreload
%autoreload 2
from datetime import datetime
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits.clickup.toolkit import ClickupToolkit
from langchain_community.utilities.clickup import ClickupAPIWrapper
from langchain_openai import OpenAI
```

## 初始化

### 获取认证

1. 创建一个 [ClickUp 应用](https://help.clickup.com/hc/en-us/articles/6303422883095-Create-your-own-app-with-the-ClickUp-API)

2. 按照 [这些步骤](https://clickup.com/api/developer-portal/authentication/) 获取你的 `client_id` 和 `client_secret`。

   - *建议：将 `https://google.com` 作为重定向 URI。这是我们在默认设置中假设的。*

3. 复制/粘贴它们，并运行下一个单元格以获取你的 `code`

```python
# Copilot Sandbox
oauth_client_id = "ABC..."
oauth_client_secret = "123..."
redirect_uri = "https://google.com"
print("点击此链接，选择你的工作区，点击“连接工作区”")
print(ClickupAPIWrapper.get_access_code_url(oauth_client_id, redirect_uri))
```

```output
点击此链接，选择你的工作区，点击“连接工作区”
https://app.clickup.com/api?client_id=ABC...&redirect_uri=https://google.com
```

URL 应该会变成类似这样的 https://www.google.com/?code=THISISMYCODERIGHTHERE。

接下来，在下面的单元格中复制/粘贴生成的 `CODE`（THISISMYCODERIGHTHERE）。

```python
code = "THISISMYCODERIGHTHERE"
```

### 获取访问令牌

然后，使用下面的代码获取你的 `access_token`。

*重要提示*：每个代码是一次性的，使用后将过期。`access_token` 可以在一段时间内使用。确保在获取到它后立即复制粘贴 `access_token`！

```python
access_token = ClickupAPIWrapper.get_access_token(
    oauth_client_id, oauth_client_secret, code
)
```

```output
错误：{'err': 'Code already used', 'ECODE': 'OAUTH_014'}
你已经使用过这个代码了。返回上一步并生成一个新的代码。
我们猜测获取新代码的 URL 是：
https://app.clickup.com/api?client_id=B5D61F8EVO04PR0JX0U73984LLS9GI6P&redirect_uri=https://google.com
```

```python
# 初始化工具包
clickup_api_wrapper = ClickupAPIWrapper(access_token=access_token)
toolkit = ClickupToolkit.from_clickup_api_wrapper(clickup_api_wrapper)
print(f"找到团队 ID：{clickup_api_wrapper.team_id}。\n大多数请求需要团队 ID，所以我们在工具包中为您存储它，我们假设您的列表中的第一个团队是您想要的团队。\n注意：如果您知道这是错误的 ID，可以在初始化时传递它。")
```

```output
找到团队 ID：9011010153。
大多数请求需要团队 ID，所以我们在工具包中为您存储它，我们假设您的列表中的第一个团队是您想要的团队。
注意：如果您知道这是错误的 ID，可以在初始化时传递它。
```

### 创建 Agent

```python
llm = OpenAI(temperature=0, openai_api_key="")
agent = initialize_agent(
    toolkit.get_tools(), llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

## 使用 Agent

```python
# 辅助函数用于演示
def print_and_run(command):
    print("\033[94m$ COMMAND\033[0m")
    print(command)
    print("\n\033[94m$ AGENT\033[0m")
    response = agent.run(command)
    print("".join(["-"] * 80))
    return response
```

### 导航

您可以获取用户可以访问的团队、文件夹和空间。

```python
print_and_run("获取用户被授权访问的所有团队")
print_and_run("获取团队可用的所有空间")
print_and_run("获取团队的所有文件夹")
```

```output
$ COMMAND
获取用户被授权访问的所有团队
$ AGENT
> 进入新的 AgentExecutor 链...
我需要使用 Get Teams 工具
操作：获取团队
操作输入：不需要的请求参数
观察结果：{'teams': [{'id': '9011010153', 'name': 'Task Copilot Sandbox Workspace 1', 'members': [{'id': 61681706, 'username': 'Aiswarya ', 'email': 'asankar@clickup.com', 'initials': 'A'}, {'id': 81928627, 'username': 'Rodrigo Ceballos Lentini', 'email': 'rlentini@clickup.com', 'initials': 'RL'}]}]}
思考：我现在知道用户被授权访问的团队了
最终答案：用户被授权访问团队 'Task Copilot Sandbox Workspace 1'。
> 完成链。
--------------------------------------------------------------------------------
$ COMMAND
获取团队可用的所有空间
$ AGENT
> 进入新的 AgentExecutor 链...
我需要使用 API 来获取空间
操作：获取团队
操作输入：不需要的请求参数
观察结果：{'teams': [{'id': '9011010153', 'name': 'Task Copilot Sandbox Workspace 1', 'members': [{'id': 61681706, 'username': 'Aiswarya ', 'email': 'asankar@clickup.com', 'initials': 'A'}, {'id': 81928627, 'username': 'Rodrigo Ceballos Lentini', 'email': 'rlentini@clickup.com', 'initials': 'RL'}]}]}
思考：我现在有了团队列表
最终答案：团队可用的空间列表在观察结果中。
> 完成链。
--------------------------------------------------------------------------------
$ COMMAND
获取团队的所有文件夹
$ AGENT
> 进入新的 AgentExecutor 链...
我需要获取团队
```

团队中的文件夹在观察中列出。

### 任务操作

您可以获取、提问有关任务并更新它们

```python
task_id = "8685mb5fn"
```

#### 基本属性获取和更新

```python
# 我们可以获取一个任务以检查其内容
print_and_run(f"获取id为{task_id}的任务")
# 我们可以从任务中获取特定属性
previous_description = print_and_run(
    f"任务id为{task_id}的描述是什么"
)
# 我们甚至可以更新它！
print_and_run(
    f"对于id为{task_id}的任务，将描述更改为'一个被AI改变的很酷的任务描述！'"
)
print_and_run(f"任务id为{task_id}的描述是什么")
# 撤消我们所做的更改
print_and_run(
    f"对于id为{task_id}的任务，将描述更改为'{previous_description}'"
)
```

```
任务8685mb5fn的描述已更新为'一个老旧、乏味的任务描述'。
```

```python
print_and_run("将任务8685mj6cd的描述更改为'看啊，没有手！'")
```

```
"任务8685mb5fn的描述已更新为'一个老旧、乏味的任务描述'。"
```

我需要更新一个任务的描述。

操作：更新任务

操作输入：{"task_id": "8685mj6cd", "attribute_name": "description", "value": "Look ma no hands"}

观察： <Response [200]>

思考：任务描述已成功更新

最终答案：任务 8685mj6cd 的描述已更改为“Look ma no hands”。

> 链结束。

--------------------------------------------------------------------------------

```output
"任务 8685mj6cd 的描述已更改为 'Look ma no hands'."
```

#### 高级属性（受让人）

您可以查询和更新几乎关于任务的所有内容！

```python
user_id = 81928627
```

```python
print_and_run(f"任务 ID {task_id} 的受让人是谁？")
print_and_run(f"从任务 ID {task_id} 的受让人中移除用户 {user_id}")
print_and_run(f"任务 ID {task_id} 的受让人是谁？")
print_and_run(f"将用户 {user_id} 添加到任务 ID {task_id} 的受让人中")
```

```output
$ COMMAND
任务 ID 8685mb5fn 的受让人是谁？
$ AGENT
> 进入新的 AgentExecutor 链...
我需要获取任务的受让人
操作：获取任务属性
操作输入：{"task_id": "8685mb5fn", "attribute_name": "assignee"}
观察：错误：在任务键 dict_keys(['id', 'name', 'text_content', 'description', 'status', 'creator_id', 'creator_username', 'creator_email', 'assignees', 'watcher_username', 'watcher_email', 'priority', 'due_date', 'start_date', 'points', 'team_id', 'project_id']) 中找不到 attribute_name = assignee。请再次调用并使用其中一个键名。
思考：我需要获取任务的受让人
操作：获取任务属性
操作输入：{"task_id": "8685mb5fn", "attribute_name": "assignees"}
观察：[]
思考：我现在知道最终答案
最终答案：任务 ID 8685mb5fn 没有受让人。
> 链结束。
--------------------------------------------------------------------------------
$ COMMAND
从任务 ID 8685mb5fn 的受让人中移除用户 81928627
$ AGENT
> 进入新的 AgentExecutor 链...
我需要更新任务的受让人
操作：更新任务的受让人
操作输入：{"task_id": "8685mb5fn", "operation": "rem", "users": [81928627]}
观察： <Response [200]>
思考：用户已从任务的受让人中移除
最终答案：用户 81928627 已从任务 ID 8685mb5fn 的受让人中移除。
> 链结束。
--------------------------------------------------------------------------------
$ COMMAND
任务 ID 8685mb5fn 的受让人是谁？
$ AGENT
> 进入新的 AgentExecutor 链...
我需要获取任务的受让人
操作：获取任务属性
操作输入：{"task_id": "8685mb5fn", "attribute_name": "assignee"}
观察：错误：在任务键 dict_keys(['id', 'name', 'text_content', 'description', 'status', 'creator_id', 'creator_username', 'creator_email', 'assignees', 'watcher_username', 'watcher_email', 'priority', 'due_date', 'start_date', 'points', 'team_id', 'project_id']) 中找不到 attribute_name = assignee。请再次调用并使用其中一个键名。
思考：我需要获取任务的受让人
操作：获取任务属性
操作输入：{"task_id": "8685mb5fn", "attribute_name": "assignees"}
观察：[]
思考：我现在知道最终答案
最终答案：任务 ID 8685mb5fn 没有受让人。
> 链结束。
--------------------------------------------------------------------------------
$ COMMAND
将用户 81928627 添加到任务 ID 8685mb5fn 的受让人中
$ AGENT
> 进入新的 AgentExecutor 链...
我需要更新任务的受让人
操作：更新任务的受让人
操作输入：{"task_id": "8685mb5fn", "operation": "rem", "users": [81928627]}
观察： <Response [200]>
思考：用户已从任务的受让人中移除
最终答案：用户 81928627 已从任务 ID 8685mb5fn 的受让人中移除。
> 链结束。
--------------------------------------------------------------------------------
```

```output
'用户 81928627 已从任务 ID 8685mb5fn 的受让人中移除。'
```

### 创建

您可以创建任务、列表和文件夹

```python
time_str = datetime.now().strftime("%d/%m/%Y-%H:%M:%S")
print_and_run(
    f"创建一个名为 'Test Task - {time_str}' 的任务，描述为 'This is a Test'"
)
```

```output
$ COMMAND
创建一个名为 'Test Task - 18/09/2023-10:31:22' 的任务，描述为 'This is a Test'
$ AGENT
> 进入新的 AgentExecutor 链...
我需要使用创建任务工具
操作：创建任务
操作输入：{"name": "Test Task - 18/09/2023-10:31:22", "description": "This is a Test"}
观察: {'id': '8685mw4wq', 'custom_id': None, 'name': 'Test Task - 18/09/2023-10:31:22', 'text_content': 'This is a Test', 'description': 'This is a Test', 'status': {'id': 'p90110061901_VlN8IJtk', 'status': 'to do', 'color': '#87909e', 'orderindex': 0, 'type': 'open'}, 'orderindex': '23.00000000000000000000000000000000', 'date_created': '1695047486396', 'date_updated': '1695047486396', 'date_closed': None, 'date_done': None, 'archived': False, 'creator': {'id': 81928627, 'username': 'Rodrigo Ceballos Lentini', 'color': '#c51162', 'email': 'rlentini@clickup.com', 'profilePicture': None}, 'assignees': [], 'watchers': [{'id': 81928627, 'username': 'Rodrigo Ceballos Lentini', 'color': '#c51162', 'initials': 'RL', 'email': 'rlentini@clickup.com', 'profilePicture': None}], 'checklists': [], 'tags': [], 'parent': None, 'priority': None, 'due_date': None, 'start_date': None, 'points': None, 'time_estimate': None, 'time_spent': 0, 'custom_fields': [], 'dependencies': [], 'linked_tasks': [], 'team_id': '9011010153', 'url': 'https://app.clickup.com/t/8685mw4wq', 'sharing': {'public': False, 'public_share_expires_on': None, 'public_fields': ['assignees', 'priority', 'due_date', 'content', 'comments', 'attachments', 'customFields', 'subtasks', 'tags', 'checklists', 'coverimage'], 'token': None, 'seo_optimized': False}, 'permission_level': 'create', 'list': {'id': '901100754275', 'name': 'Test List', 'access': True}, 'project': {'id': '90110336890', 'name': 'Test Folder', 'hidden': False, 'access': True}, 'folder': {'id': '90110336890', 'name': 'Test Folder', 'hidden': False, 'access': True}, 'space': {'id': '90110061901'}}
想法: 我现在知道最终答案
最终答案: 成功创建了一个名为“Test Task - 18/09/2023-10:31:22”的任务，描述为“这是一个测试”。
> 完成链。
```

```output
"成功创建了一个名为“Test Task - 18/09/2023-10:31:22”的任务，描述为“这是一个测试”。
```

```python
time_str = datetime.now().strftime("%d/%m/%Y-%H:%M:%S")
print_and_run(f"创建一个名为Test List - {time_str}的列表")
```

```output
$ COMMAND
创建一个名为Test List - 18/09/2023-10:32:12的列表
$ AGENT
> 进入新的 AgentExecutor 链...
 我需要创建一个列表
动作: 创建列表
动作输入: {"name": "Test List - 18/09/2023-10:32:12"}
观察: {'id': '901100774700', 'name': 'Test List - 18/09/2023-10:32:12', 'deleted': False, 'orderindex': 13, 'content': '', 'priority': None, 'assignee': None, 'due_date': None, 'start_date': None, 'folder': {'id': '90110336890', 'name': 'Test Folder', 'hidden': False, 'access': True}, 'space': {'id': '90110061901', 'name': 'Space', 'access': True}, 'inbound_address': 'a.t.901100774700.u-81928627.20b87d50-eece-4721-b487-9ca500338587@tasks.clickup.com', 'archived': False, 'override_statuses': False, 'statuses': [{'id': 'p90110061901_VlN8IJtk', 'status': 'to do', 'orderindex': 0, 'color': '#87909e', 'type': 'open'}, {'id': 'p90110061901_14GpYKnM', 'status': 'complete', 'orderindex': 1, 'color': '#6bc950', 'type': 'closed'}], 'permission_level': 'create'}
想法: 我现在知道最终答案
最终答案: 已成功创建了名为“Test List - 18/09/2023-10:32:12”的列表，其 id 为 901100774700。
> 完成链。
```

```output
'已成功创建了名为“Test List - 18/09/2023-10:32:12”的列表，其 id 为 901100774700。'
```

```python
time_str = datetime.now().strftime("%d/%m/%Y-%H:%M:%S")
print_and_run(f"创建一个名为'Test Folder - {time_str}'的文件夹")
```

```output
$ COMMAND
创建一个名为'Test Folder - 18/09/2023-10:32:51'的文件夹
$ AGENT
> 进入新的 AgentExecutor 链...
 我需要使用创建文件夹工具
动作: 创建文件夹
动作输入: {"name": "Test Folder - 18/09/2023-10:32:51"}
观察: {'id': '90110348711', 'name': 'Test Folder - 18/09/2023-10:32:51', 'orderindex': 12, 'override_statuses': False, 'hidden': False, 'space': {'id': '90110061901', 'name': 'Space', 'access': True}, 'task_count': '0', 'archived': False, 'statuses': [], 'lists': [], 'permission_level': 'create'}
想法: 我已成功创建了文件夹
最终答案: 成功创建了一个名为“Test Folder - 18/09/2023-10:32:51”的文件夹。
> 完成链。
```

```output
"成功创建了一个名为“Test Folder - 18/09/2023-10:32:51”的文件夹。"
```

```python
time_str = datetime.now().strftime("%d/%m/%Y-%H:%M:%S")
print_and_run(
    f"创建一个名为'Test List - {time_str}'的列表，内容为我的测试列表，优先级高，状态为红色"
)
```

```output
$ COMMAND
创建一个名为'Test List - 18/09/2023-10:34:01'的列表，内容为我的测试列表，优先级高，状态为红色
$ AGENT
```

```markdown
> 进入新的 AgentExecutor 链...
我需要创建一个包含名称、内容、优先级和状态的列表
动作：创建列表
动作输入：{"name": "Test List - 18/09/2023-10:34:01", "content": "My test list", "priority": 2, "status": "red"}
观察：{'id': '901100774746', 'name': 'Test List - 18/09/2023-10:34:01', 'deleted': False, 'orderindex': 15, 'content': '', 'status': {'status': 'red', 'color': '#e50000', 'hide_label': True}, 'priority': {'priority': 'high', 'color': '#ffcc00'}, 'assignee': None, 'due_date': None, 'start_date': None, 'folder': {'id': '90110336890', 'name': 'Test Folder', 'hidden': False, 'access': True}, 'space': {'id': '90110061901', 'name': 'Space', 'access': True}, 'inbound_address': 'a.t.901100774746.u-81928627.2ab87133-728e-4166-b2ae-423cc320df37@tasks.clickup.com', 'archived': False, 'override_statuses': False, 'statuses': [{'id': 'p90110061901_VlN8IJtk', 'status': 'to do', 'orderindex': 0, 'color': '#87909e', 'type': 'open'}, {'id': 'p90110061901_14GpYKnM', 'status': 'complete', 'orderindex': 1, 'color': '#6bc950', 'type': 'closed'}], 'permission_level': 'create'}
想法：我已成功创建了列表
最终答案："Test List - 18/09/2023-10:34:01" 的列表，内容为 "My test list"，优先级高，状态为红色，已成功创建。
> 链结束。
--------------------------------------------------------------------------------
```

```output
"The list 'Test List - 18/09/2023-10:34:01' with content 'My test list' with high priority and status red has been successfully created."
```

## 多步任务

```python
print_and_run(
    "找出 Rodrigo 的用户 ID，创建一个名为 'Rod's task' 的任务，并将其分配给 Rodrigo"
)
```

```output
$ COMMAND
找出 Rodrigo 的用户 ID，创建一个名为 'Rod's task' 的任务，并将其分配给 Rodrigo
$ AGENT
> 进入新的 AgentExecutor 链...
我需要获取 Rodrigo 的用户 ID，创建一个任务，并将其分配给 Rodrigo
动作：获取团队
动作输入：无需输入
观察：{'teams': [{'id': '9011010153', 'name': 'Task Copilot Sandbox Workspace 1', 'members': [{'id': 81928627, 'username': 'Rodrigo Ceballos Lentini', 'email': 'rlentini@clickup.com', 'initials': 'RL'}]}]}
想法：我现在已经获得了 Rodrigo 的用户 ID
动作：创建任务
动作输入：{"name": "Rod's task", "assignees": [81928627]}
```

```output
/Users/rodrigolentini/repos/langchain-clickup/libs/langchain/langchain/utilities/clickup.py:145: UserWarning: 在尝试解析 <class 'langchain_community.utilities/clickup.Task'> 时遇到错误：'NoneType' object is not subscriptable
退回到返回输入数据。
  warnings.warn(f'Error encountered while trying to parse {dataclass}: {e}\n Falling back to returning input data.')
```

```output
观察：{'id': '8685mw6dz', 'custom_id': None, 'name': "Rod's task", 'text_content': '', 'description': '', 'status': {'id': 'p90110061901_VlN8IJtk', 'status': 'to do', 'color': '#87909e', 'orderindex': 0, 'type': 'open'}, 'orderindex': '24.00000000000000000000000000000000', 'date_created': '1695047740939', 'date_updated': '1695047740939', 'date_closed': None, 'date_done': None, 'archived': False, 'creator': {'id': 81928627, 'username': 'Rodrigo Ceballos Lentini', 'color': '#c51162', 'email': 'rlentini@clickup.com', 'profilePicture': None}, 'assignees': [{'id': 81928627, 'username': 'Rodrigo Ceballos Lentini', 'color': '#c51162', 'initials': 'RL', 'email': 'rlentini@clickup.com', 'profilePicture': None}], 'watchers': [{'id': 81928627, 'username': 'Rodrigo Ceballos Lentini', 'color': '#c51162', 'initials': 'RL', 'email': 'rlentini@clickup.com', 'profilePicture': None}], 'checklists': [], 'tags': [], 'parent': None, 'priority': None, 'due_date': None, 'start_date': None, 'points': None, 'time_estimate': None, 'time_spent': 0, 'custom_fields': [], 'dependencies': [], 'linked_tasks': [], 'team_id': '9011010153', 'url': 'https://app.clickup.com/t/8685mw6dz', 'sharing': {'public': False, 'public_share_expires_on': None, 'public_fields': ['assignees', 'priority', 'due_date', 'content', 'comments', 'attachments', 'customFields', 'subtasks', 'tags', 'checklists', 'coverimage'], 'token': None, 'seo_optimized': False}, 'permission_level': 'create', 'list': {'id': '901100754275', 'name': 'Test List', 'access': True}, 'project': {'id': '90110336890', 'name': 'Test Folder', 'hidden': False, 'access': True}, 'folder': {'id': '90110336890', 'name': 'Test Folder', 'hidden': False, 'access': True}, 'space': {'id': '90110061901'}}
想法：我现在已经创建了任务并分配给 Rodrigo
最终答案：Rodrigo 的用户 ID 是 81928627，名为 'Rod's task' 的任务已成功创建并分配给他。
> 链结束。
--------------------------------------------------------------------------------
```

```output
"Rodrigo's user ID is 81928627 and a task called 'Rod's task' has been created and assigned to him."
```
