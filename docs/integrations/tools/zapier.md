# Zapier 自然语言操作

**已弃用** 该 API 将在 2023-11-17 日停用：https://nla.zapier.com/start/

[Zapier 自然语言操作](https://nla.zapier.com/start/) 通过自然语言 API 接口为您提供对 Zapier 平台上 5000 多个应用和 20000 多个操作的访问权限。

NLA 支持诸如 `Gmail`、`Salesforce`、`Trello`、`Slack`、`Asana`、`HubSpot`、`Google Sheets`、`Microsoft Teams` 等成千上万个应用：https://zapier.com/apps

`Zapier NLA` 处理所有底层 API 认证和自然语言 --> 底层 API 调用 --> 返回简化输出给 LLMs。其关键思想是您或您的用户通过类似 OAuth 的设置窗口公开一组操作，然后可以通过 REST API 进行查询和执行。

NLA 提供 API 密钥和 OAuth 两种方式来签署 NLA API 请求。

1. 服务器端（API 密钥）：用于快速入门、测试和生产场景，LangChain 仅使用开发者在 Zapier.com 上连接的账户中公开的操作。

2. 用户界面（OAuth）：用于部署面向最终用户的应用程序的生产场景，LangChain 需要访问最终用户在 Zapier.com 上公开的操作和连接的账户。

出于简洁起见，本快速入门主要关注服务器端使用情况。请跳转至[使用 OAuth 访问令牌示例](#oauth)查看如何设置 Zapier 以用于用户界面情况的简短示例。请查看[完整文档](https://nla.zapier.com/start/)以获取完整的面向用户的 OAuth 开发者支持。

本示例介绍了如何使用 Zapier 集成与 `SimpleSequentialChain`，然后是 `Agent`。

在下面的代码中：

```python
import os
# 从 https://platform.openai.com/ 获取
os.environ["OPENAI_API_KEY"] = os.environ.get("OPENAI_API_KEY", "")
# 从 https://nla.zapier.com/docs/authentication/ 登录后获取：
os.environ["ZAPIER_NLA_API_KEY"] = os.environ.get("ZAPIER_NLA_API_KEY", "")
```

## 代理示例

Zapier 工具可以与代理一起使用。请参阅下面的示例。

```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits import ZapierToolkit
from langchain_community.utilities.zapier import ZapierNLAWrapper
from langchain_openai import OpenAI
```

```python
## 步骤 0. 公开 Gmail 的“查找邮件”和 Slack 的“发送频道消息”操作
# 首先转到此处，登录，公开（启用）这两个操作：https://nla.zapier.com/demo/start -- 对于此示例，可以将所有字段留空以“让 AI 猜测”
# 在 OAuth 场景中，您将获得自己的 <provider> id（而不是 'demo'），您将首先将您的用户路由到此处
```

```python
llm = OpenAI(temperature=0)
zapier = ZapierNLAWrapper()
toolkit = ZapierToolkit.from_zapier_nla_wrapper(zapier)
agent = initialize_agent(
    toolkit.get_tools(), llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
agent.run(
    "总结我收到的有关硅谷银行的最新电子邮件。将摘要发送到 Slack 的 #test-zapier 频道。"
)
```

```output
> 进入新的 AgentExecutor 链...
我需要找到邮件并对其进行总结。
操作：Gmail：查找邮件
操作输入：查找来自硅谷银行的最新电子邮件
观察结果：{"from__name": "硅谷桥银行，N.A.", "from__email": "sreply@svb.com", "body_plain": "亲爱的客户，经过混乱、动荡和紧张的日子，我们对 SVB 的道路有了明确的认识，FDIC 完全保险所有存款，并且在我们重建时向客户和合作伙伴提出了要求。Tim Mayopoulos <https://eml.svb.com/NjEwLUtBSy0yNjYAAAGKgoxUeBCLAyF_NxON97X4rKEaNBLG", "reply_to__email": "sreply@svb.com", "subject": "认识新 CEO Tim Mayopoulos", "date": "Tue, 14 Mar 2023 23:42:29 -0500 (CDT)", "message_url": "https://mail.google.com/mail/u/0/#inbox/186e393b13cfdf0a", "attachment_count": "0", "to__emails": "ankush@langchain.dev", "message_id": "186e393b13cfdf0a", "labels": "重要，类别更新，收件箱"}
思考：我需要对电子邮件进行总结，并将其发送到 Slack 的 #test-zapier 频道。
操作：Slack：发送频道消息
操作输入：向 #test-zapier 频道发送一条 Slack 消息，内容为“硅谷银行宣布 Tim Mayopoulos 是新 CEO。FDIC 完全保险所有存款，并且在他们重建时向客户和合作伙伴提出了要求。”
观察结果：{"message__text": "硅谷银行宣布 Tim Mayopoulos 是新 CEO。FDIC 完全保险所有存款，并且在他们重建时向客户和合作伙伴提出了要求。", "message__permalink": "https://langchain.slack.com/archives/C04TSGU0RA7/p1678859932375259", "channel": "C04TSGU0RA7", "message__bot_profile__name": "Zapier", "message__team": "T04F8K3FZB5", "message__bot_id": "B04TRV4R74K", "message__bot_profile__deleted": "false", "message__bot_profile__app_id": "A024R9PQM", "ts_time": "2023-03-15T05:58:52Z", "message__bot_profile__icons__image_36": "https://avatars.slack-edge.com/2022-08-02/3888649620612_f864dc1bb794cf7d82b0_36.png", "message__blocks[]block_id": "kdZZ", "message__blocks[]elements[]type": "['rich_text_section']"}
思考：我现在知道最终答案。
最终答案：我已将硅谷银行最新邮件的摘要发送到 Slack 的 #test-zapier 频道。
> 链结束。
```

我已经将上一封来自硅谷银行的邮件摘要发送到 Slack 的 #test-zapier 频道中。

## 使用 SimpleSequentialChain 的示例

如果您需要更明确的控制，请使用下面的链条。

```python
from langchain.chains import LLMChain, SimpleSequentialChain, TransformChain
from langchain_community.tools.zapier.tool import ZapierNLARunAction
from langchain_community.utilities.zapier import ZapierNLAWrapper
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
```

```python
## 步骤 0. 公开 Gmail“查找邮件”和 Slack“发送直接消息”操作
# 首先转到此处，登录，公开（启用）这两个操作：https://nla.zapier.com/demo/start -- 对于此示例，可以将所有字段留空“Have AI guess”
# 在 OAuth 场景中，您将获得自己的 <provider> id（而不是 'demo'），您需要先将用户路由到此处
actions = ZapierNLAWrapper().list()
```

```python
## 步骤 1. Gmail 查找邮件
GMAIL_SEARCH_INSTRUCTIONS = "从硅谷银行获取最新的电子邮件"
def nla_gmail(inputs):
    action = next(
        (a for a in actions if a["description"].startswith("Gmail: Find Email")), None
    )
    return {
        "email_data": ZapierNLARunAction(
            action_id=action["id"],
            zapier_description=action["description"],
            params_schema=action["params"],
        ).run(inputs["instructions"])
    }
gmail_chain = TransformChain(
    input_variables=["instructions"],
    output_variables=["email_data"],
    transform=nla_gmail,
)
```

```python
## 步骤 2. 生成草稿回复
template = """您是一个助理，负责起草回复收到的电子邮件。以纯文本形式输出草稿回复（而非 JSON）。
收到的电子邮件：
{email_data}
草稿邮件回复："""
prompt_template = PromptTemplate(input_variables=["email_data"], template=template)
reply_chain = LLMChain(llm=OpenAI(temperature=0.7), prompt=prompt_template)
```

```python
## 步骤 3. 通过 Slack 直接消息发送草稿回复
SLACK_HANDLE = "@Ankush Gola"
def nla_slack(inputs):
    action = next(
        (
            a
            for a in actions
            if a["description"].startswith("Slack: Send Direct Message")
        ),
        None,
    )
    instructions = f'将此消息发送到 Slack 中的 {SLACK_HANDLE}：{inputs["draft_reply"]}'
    return {
        "slack_data": ZapierNLARunAction(
            action_id=action["id"],
            zapier_description=action["description"],
            params_schema=action["params"],
        ).run(instructions)
    }
slack_chain = TransformChain(
    input_variables=["draft_reply"],
    output_variables=["slack_data"],
    transform=nla_slack,
)
```

```python
## 最后，执行
overall_chain = SimpleSequentialChain(
    chains=[gmail_chain, reply_chain, slack_chain], verbose=True
)
overall_chain.run(GMAIL_SEARCH_INSTRUCTIONS)
```

```output
> 进入新的 SimpleSequentialChain 链...
{"from__name": "硅谷桥银行", "from__email": "sreply@svb.com", "body_plain": "亲爱的客户，经过混乱、动荡和紧张的日子后，我们对 SVB 的道路有了明确的认识，FDIC 完全保险所有存款，并在我们重建时向客户和合作伙伴提出了要求。Tim Mayopoulos <https://eml.svb.com/NjEwLUtBSy0yNjYAAAGKgoxUeBCLAyF_NxON97X4rKEaNBLG", "reply_to__email": "sreply@svb.com", "subject": "认识新 CEO Tim Mayopoulos", "date": "2023 年 3 月 14 日，周二，下午 11:42:29 -0500（CDT）", "message_url": "https://mail.google.com/mail/u/0/#inbox/186e393b13cfdf0a", "attachment_count": "0", "to__emails": "ankush@langchain.dev", "message_id": "186e393b13cfdf0a", "labels": "重要，类别更新，收件箱"}
亲爱的硅谷桥银行，
感谢您的电子邮件和关于您的新 CEO Tim Mayopoulos 的更新。我们感谢您致力于让您的客户和合作伙伴保持了解，并期待继续与您保持合作关系。
此致，
[您的名字]
{"message__text": "亲爱的硅谷桥银行，\n\n感谢您的电子邮件和关于您的新 CEO Tim Mayopoulos 的更新。我们感谢您致力于让您的客户和合作伙伴保持了解，并期待继续与您保持合作关系。\n\n此致，\n\n[您的名字]", "message__permalink": "https://langchain.slack.com/archives/D04TKF5BBHU/p1678859968241629", "channel": "D04TKF5BBHU", "message__bot_profile__name": "Zapier", "message__team": "T04F8K3FZB5", "message__bot_id": "B04TRV4R74K", "message__bot_profile__deleted": "false", "message__bot_profile__app_id": "A024R9PQM", "ts_time": "2023-03-15T05:59:28Z", "message__blocks[]block_id": "p7i", "message__blocks[]elements[]elements[]type": "[['text']]", "message__blocks[]elements[]type": "['rich_text_section']"}
> 链条执行完毕。
```

## <a id="oauth">使用 OAuth 访问令牌的示例</a>

以下代码片段显示了如何使用获取的 OAuth 访问令牌初始化包装器。请注意传递的参数，而不是设置环境变量。请查看[认证文档](https://nla.zapier.com/docs/authentication/#oauth-credentials)以获取完整的面向用户的 OAuth 开发者支持。

开发人员的任务是处理 OAuth 握手以获取和刷新访问令牌。

```python
llm = OpenAI(temperature=0)
zapier = ZapierNLAWrapper(zapier_nla_oauth_access_token="<填写访问令牌>")
toolkit = ZapierToolkit.from_zapier_nla_wrapper(zapier)
agent = initialize_agent(
    toolkit.get_tools(), llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
agent.run(
    "总结我收到的有关硅谷银行的最后一封电子邮件。将摘要发送到 Slack 的 #test-zapier 频道。"
)
```