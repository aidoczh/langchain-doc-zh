# Oracle 自主数据库

Oracle自主数据库是一种云数据库，利用机器学习来自动化数据库调优、安全性、备份、更新以及其他传统由数据库管理员（DBAs）执行的例行管理任务。

本文档介绍了如何从Oracle自主数据库加载文档，加载器支持使用连接字符串或TNS配置进行连接。

## 先决条件

1. 数据库以“Thin”模式运行：

   https://python-oracledb.readthedocs.io/en/latest/user_guide/appendix_b.html

2. `pip install oracledb`：

   https://python-oracledb.readthedocs.io/en/latest/user_guide/installation.html

## 指令

```python
pip install oracledb
```

```python
from langchain_community.document_loaders import OracleAutonomousDatabaseLoader
from settings import s
```

使用双向 TLS 认证（mTLS），需要提供 wallet_location 和 wallet_password 来创建连接，用户可以通过提供连接字符串或 TNS 配置详细信息来创建连接。

```python
SQL_QUERY = "select prod_id, time_id from sh.costs fetch first 5 rows only"
doc_loader_1 = OracleAutonomousDatabaseLoader(
    query=SQL_QUERY,
    user=s.USERNAME,
    password=s.PASSWORD,
    schema=s.SCHEMA,
    config_dir=s.CONFIG_DIR,
    wallet_location=s.WALLET_LOCATION,
    wallet_password=s.PASSWORD,
    tns_name=s.TNS_NAME,
)
doc_1 = doc_loader_1.load()
doc_loader_2 = OracleAutonomousDatabaseLoader(
    query=SQL_QUERY,
    user=s.USERNAME,
    password=s.PASSWORD,
    schema=s.SCHEMA,
    connection_string=s.CONNECTION_STRING,
    wallet_location=s.WALLET_LOCATION,
    wallet_password=s.PASSWORD,
)
doc_2 = doc_loader_2.load()
```

使用 TLS 认证时，不需要 wallet_location 和 wallet_password。

```python
doc_loader_3 = OracleAutonomousDatabaseLoader(
    query=SQL_QUERY,
    user=s.USERNAME,
    password=s.PASSWORD,
    schema=s.SCHEMA,
    config_dir=s.CONFIG_DIR,
    tns_name=s.TNS_NAME,
)
doc_3 = doc_loader_3.load()
doc_loader_4 = OracleAutonomousDatabaseLoader(
    query=SQL_QUERY,
    user=s.USERNAME,
    password=s.PASSWORD,
    schema=s.SCHEMA,
    connection_string=s.CONNECTION_STRING,
)
doc_4 = doc_loader_4.load()
```