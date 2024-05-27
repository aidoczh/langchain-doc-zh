# AINetwork

[AI Network](https://www.ainetwork.ai/build-on-ain) 是一个设计用于容纳大规模AI模型的一级区块链，利用由 [$AIN token](https://www.ainetwork.ai/token) 驱动的去中心化GPU网络，丰富了基于AI的 `NFTs` (`AINFTs`)。

`AINetwork Toolkit` 是一组用于与 [AINetwork Blockchain](https://www.ainetwork.ai/public/whitepaper.pdf) 交互的工具。这些工具允许您转移 `AIN`，读取和写入数值，创建应用程序，并为区块链数据库中的特定路径设置权限。

## 安装依赖

在使用 AINetwork Toolkit 之前，您需要安装 ain-py 包。您可以使用 pip 安装它：

```python
%pip install --upgrade --quiet  ain-py
```

## 设置环境变量

您需要将 `AIN_BLOCKCHAIN_ACCOUNT_PRIVATE_KEY` 环境变量设置为您的 AIN 区块链账户私钥。

```python
import os
os.environ["AIN_BLOCKCHAIN_ACCOUNT_PRIVATE_KEY"] = ""
```

### 获取 AIN 区块链私钥

```python
import os
from ain.account import Account
if os.environ.get("AIN_BLOCKCHAIN_ACCOUNT_PRIVATE_KEY", None):
    account = Account(os.environ["AIN_BLOCKCHAIN_ACCOUNT_PRIVATE_KEY"])
else:
    account = Account.create()
    os.environ["AIN_BLOCKCHAIN_ACCOUNT_PRIVATE_KEY"] = account.private_key
    print(
        f"""
address: {account.address}
private_key: {account.private_key}
"""
    )
# 重要提示：如果您打算将来使用此账户，请确保将私钥保存在安全的地方。丢失私钥意味着丢失对您的账户的访问权限。
```

```output
address: 0x5BEB4Defa2ccc274498416Fd7Cb34235DbC122Ac
private_key: f5e2f359bb6b7836a2ac70815473d1a290c517f847d096f5effe818de8c2cf14
```

## 初始化 AINetwork Toolkit

您可以像这样初始化 AINetwork Toolkit：

```python
from langchain_community.agent_toolkits.ainetwork.toolkit import AINetworkToolkit
toolkit = AINetworkToolkit()
tools = toolkit.get_tools()
address = tools[0].interface.wallet.defaultAccount.address
```

## 使用 AINetwork Toolkit 初始化 Agent

您可以像这样使用 AINetwork Toolkit 初始化 Agent：

```python
from langchain.agents import AgentType, initialize_agent
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(temperature=0)
agent = initialize_agent(
    tools=tools,
    llm=llm,
    verbose=True,
    agent=AgentType.OPENAI_FUNCTIONS,
)
```

## 示例用法

以下是您可以如何使用 AINetwork Toolkit 与 Agent 的一些示例：

### 定义要测试的应用程序名称

```python
appName = f"langchain_demo_{address.lower()}"
```

### 在 AINetwork 区块链数据库中创建应用程序

```python
print(
    agent.run(
        f"Create an app in the AINetwork Blockchain database with the name {appName}"
    )
)
```

```output
> 进入新的 AgentExecutor 链...
调用: `AINappOps` with `{'type': 'SET_ADMIN', 'appName': 'langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac'}`
{"tx_hash": "0x018846d6a9fc111edb1a2246ae2484ef05573bd2c584f3d0da155fa4b4936a9e", "result": {"gas_amount_total": {"bandwidth": {"service": 4002, "app": {"langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac": 2}}, "state": {"service": 1640}}, "gas_cost_total": 0, "func_results": {"_createApp": {"op_results": {"0": {"path": "/apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac", "result": {"code": 0, "bandwidth_gas_amount": 1}}, "1": {"path": "/apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac", "result": {"code": 0, "bandwidth_gas_amount": 1}}, "2": {"path": "/manage_app/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac/config/admin", "result": {"code": 0, "bandwidth_gas_amount": 1}}}, "code": 0, "bandwidth_gas_amount": 2000}}, "code": 0, "bandwidth_gas_amount": 2001, "gas_amount_charged": 5642}}
已在 AINetwork 区块链数据库中创建了名称为 "langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac" 的应用程序。
> 链结束。
已在 AINetwork 区块链数据库中创建了名称为 "langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac" 的应用程序。
```

### 在 AINetwork 区块链数据库中的特定路径设置值

```python
print(
    agent.run(f"Set the value {{1: 2, '34': 56}} at the path /apps/{appName}/object .")
)
```

```output
> 进入新的 AgentExecutor 链...
调用: `AINvalueOps` with `{'type': 'SET', 'path': '/apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac/object', 'value': {'1': 2, '34': 56}}`
{"tx_hash": "0x3d1a16d9808830088cdf4d37f90f4b1fa1242e2d5f6f983829064f45107b5279", "result": {"gas_amount_total": {"bandwidth": {"service": 0, "app": {"langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac": 1}}, "state": {"service": 0, "app": {"langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac": 674}}}, "gas_cost_total": 0, "code": 0, "bandwidth_gas_amount": 1, "gas_amount_charged": 0}}
已在路径 /apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac/object 设置了值 {1: 2, '34': 56}。
> 链结束。
已在路径 /apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac/object 设置了值 {1: 2, '34': 56}。
```

### 在 AINetwork 区块链数据库中为路径设置权限

```python
print(
    agent.run(
        f"设置路径 /apps/{appName}/user/$from 的写入权限，使用评估字符串 auth.addr===$from。"
    )
)
```

```output
> 进入新的 AgentExecutor 链...
调用：`AINruleOps`，参数为 `{'type': 'SET', 'path': '/apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac/user/$from', 'eval': 'auth.addr===$from'}`
{"tx_hash": "0x37d5264e580f6a217a347059a735bfa9eb5aad85ff28a95531c6dc09252664d2", "result": {"gas_amount_total": {"bandwidth": {"service": 0, "app": {"langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac": 1}}, "state": {"service": 0, "app": {"langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac": 712}}}, "gas_cost_total": 0, "code": 0, "bandwidth_gas_amount": 1, "gas_amount_charged": 0}}
路径 `/apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac/user/$from` 的写入权限已设置，评估字符串为 `auth.addr===$from`。
> 链结束。
路径 `/apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac/user/$from` 的写入权限已设置，评估字符串为 `auth.addr===$from`。
```

### 获取 AINetwork 区块链数据库中路径的权限

```python
print(agent.run(f"获取路径 /apps/{appName} 的权限。"))
```

```output
> 进入新的 AgentExecutor 链...
调用：`AINownerOps`，参数为 `{'type': 'GET', 'path': '/apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac'}`
{".owner": {"owners": {"0x5BEB4Defa2ccc274498416Fd7Cb34235DbC122Ac": {"branch_owner": true, "write_function": true, "write_owner": true, "write_rule": true}}}}
路径 /apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac 的权限如下：
- 地址：0x5BEB4Defa2ccc274498416Fd7Cb34235DbC122Ac
  - branch_owner: true
  - write_function: true
  - write_owner: true
  - write_rule: true
> 链结束。
路径 /apps/langchain_demo_0x5beb4defa2ccc274498416fd7cb34235dbc122ac 的权限如下：
- 地址：0x5BEB4Defa2ccc274498416Fd7Cb34235DbC122Ac
  - branch_owner: true
  - write_function: true
  - write_owner: true
  - write_rule: true
```

### 从水龙头获取 AIN

```python
!curl http://faucet.ainetwork.ai/api/test/{address}/
```

```output
{"result":"0x0eb07b67b7d0a702cb60e865d3deafff3070d8508077ef793d69d6819fd92ea3","time":1692348112376}
```

### 获取 AIN 余额

```python
print(agent.run(f"检查地址 {address} 的 AIN 余额"))
```

```output
> 进入新的 AgentExecutor 链...
调用：`AINvalueOps`，参数为 `{'type': 'GET', 'path': '/accounts/0x5BEB4Defa2ccc274498416Fd7Cb34235DbC122Ac/balance'}`
100
地址 0x5BEB4Defa2ccc274498416Fd7Cb34235DbC122Ac 的 AIN 余额为 100。
> 链结束。
地址 0x5BEB4Defa2ccc274498416Fd7Cb34235DbC122Ac 的 AIN 余额为 100 AIN。
```

### 转账 AIN

```python
print(
    agent.run(
        "向地址 0x19937b227b1b13f29e7ab18676a89ea3bdea9c5b 转账 100 AIN"
    )
)
```

```output
> 进入新的 AgentExecutor 链...
调用：`AINtransfer`，参数为 `{'address': '0x19937b227b1b13f29e7ab18676a89ea3bdea9c5b', 'amount': 100}`
{"tx_hash": "0xa59d15d23373bcc00e413ac8ba18cb016bb3bdd54058d62606aec688c6ad3d2e", "result": {"gas_amount_total": {"bandwidth": {"service": 3}, "state": {"service": 866}}, "gas_cost_total": 0, "func_results": {"_transfer": {"op_results": {"0": {"path": "/accounts/0x5BEB4Defa2ccc274498416Fd7Cb34235DbC122Ac/balance", "result": {"code": 0, "bandwidth_gas_amount": 1}}, "1": {"path": "/accounts/0x19937B227b1b13f29e7AB18676a89EA3BDEA9C5b/balance", "result": {"code": 0, "bandwidth_gas_amount": 1}}}, "code": 0, "bandwidth_gas_amount": 0}}, "code": 0, "bandwidth_gas_amount": 1, "gas_amount_charged": 869}}
向地址 0x19937b227b1b13f29e7ab18676a89ea3bdea9c5b 转账 100 AIN 成功。交易哈希为 0xa59d15d23373bcc00e413ac8ba18cb016bb3bdd54058d62606aec688c6ad3d2e。
> 链结束。
向地址 0x19937b227b1b13f29e7ab18676a89ea3bdea9c5b 转账 100 AIN 成功。交易哈希为 0xa59d15d23373bcc00e413ac8ba18cb016bb3bdd54058d62606aec688c6ad3d2e。
```