# Dataherald

本文介绍如何使用 Dataherald 组件。

首先，您需要设置 Dataherald 账户并获取 API 密钥：

1. 前往 dataherald 网站并注册 [链接](https://www.dataherald.com/)

2. 登录管理控制台后，创建一个 API 密钥

3. 运行以下命令安装 dataherald：

```python
pip install dataherald
```

接下来，我们需要设置一些环境变量：

1. 将您的 API 密钥保存到 DATAHERALD_API_KEY 环境变量中

```python
import os
os.environ["DATAHERALD_API_KEY"] = ""
```

然后，我们可以使用以下代码连接到 Dataherald API：

```python
from langchain_community.utilities.dataherald import DataheraldAPIWrapper
dataherald = DataheraldAPIWrapper(db_connection_id="65fb766367dd22c99ce1a12d")
```

最后，我们可以运行以下代码来查询公司的员工数量：

```python
dataherald.run("How many employees are in the company?")
```

```output
'select COUNT(*) from employees'
```