# Gmail

这篇笔记将介绍如何将 LangChain 电子邮件连接到 `Gmail API`。

要使用这个工具包，你需要按照[Gmail API 文档](https://developers.google.com/gmail/api/quickstart/python#authorize_credentials_for_a_desktop_application)中的说明设置你的凭据。一旦你下载了 `credentials.json` 文件，你就可以开始使用 Gmail API 了。完成这一步之后，我们将安装所需的库。

```python
%pip install --upgrade --quiet  google-api-python-client > /dev/null
%pip install --upgrade --quiet  google-auth-oauthlib > /dev/null
%pip install --upgrade --quiet  google-auth-httplib2 > /dev/null
%pip install --upgrade --quiet  beautifulsoup4 > /dev/null # 这是可选的，但对于解析 HTML 消息很有用
```

你还需要安装 `langchain-community` 包，这是集成所在的地方。

```bash
pip install -U langchain-community
```

设置 [LangSmith](https://smith.langchain.com/) 也很有帮助（但不是必需的）。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 创建工具包

默认情况下，工具包会读取本地的 `credentials.json` 文件。你也可以手动提供一个 `Credentials` 对象。

```python
from langchain_community.agent_toolkits import GmailToolkit
toolkit = GmailToolkit()
```

### 自定义身份验证

在后台，使用以下方法创建了一个 `googleapi` 资源。你可以手动构建一个 `googleapi` 资源以获得更多的身份验证控制。

```python
from langchain_community.tools.gmail.utils import (
    build_resource_service,
    get_gmail_credentials,
)
# 可以在这里查看范围 https://developers.google.com/gmail/api/auth/scopes
# 例如，只读范围是 'https://www.googleapis.com/auth/gmail.readonly'
credentials = get_gmail_credentials(
    token_file="token.json",
    scopes=["https://mail.google.com/"],
    client_secrets_file="credentials.json",
)
api_resource = build_resource_service(credentials=credentials)
toolkit = GmailToolkit(api_resource=api_resource)
```

```python
tools = toolkit.get_tools()
tools
```

```output
[GmailCreateDraft(name='create_gmail_draft', description='Use this tool to create a draft email with the provided message fields.', args_schema=<class 'langchain_community.tools.gmail.create_draft.CreateDraftSchema'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, api_resource=<googleapiclient.discovery.Resource object at 0x10e5c6d10>),
 GmailSendMessage(name='send_gmail_message', description='Use this tool to send email messages. The input is the message, recipents', args_schema=None, return_direct=False, verbose=False, callbacks=None, callback_manager=None, api_resource=<googleapiclient.discovery.Resource object at 0x10e5c6d10>),
 GmailSearch(name='search_gmail', description=('Use this tool to search for email messages or threads. The input must be a valid Gmail query. The output is a JSON list of the requested resource.',), args_schema=<class 'langchain_community.tools.gmail.search.SearchArgsSchema'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, api_resource=<googleapiclient.discovery.Resource object at 0x10e5c6d10>),
 GmailGetMessage(name='get_gmail_message', description='Use this tool to fetch an email by message ID. Returns the thread ID, snipet, body, subject, and sender.', args_schema=<class 'langchain_community.tools.gmail.get_message.SearchArgsSchema'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, api_resource=<googleapiclient.discovery.Resource object at 0x10e5c6d10>),
 GmailGetThread(name='get_gmail_thread', description=('Use this tool to search for email messages. The input must be a valid Gmail query. The output is a JSON list of messages.',), args_schema=<class 'langchain_community.tools.gmail.get_thread.GetThreadSchema'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, api_resource=<googleapiclient.discovery.Resource object at 0x10e5c6d10>)]
```

## 使用

我们在这里展示如何将其作为[代理](/docs/tutorials/agents)的一部分使用。我们使用 OpenAI Functions 代理，因此我们需要为其设置和安装所需的依赖项。我们还将使用 [LangSmith Hub](https://smith.langchain.com/hub) 来提取提示，因此我们需要安装它。

```bash
pip install -U langchain-openai langchainhub
```

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
```

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
```

```python
instructions = """You are an assistant."""
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
```

```python
llm = ChatOpenAI(temperature=0)
```

```python
agent = create_openai_functions_agent(llm, toolkit.get_tools(), prompt)
```

```python
agent_executor = AgentExecutor(
    agent=agent,
    tools=toolkit.get_tools(),
    # This is set to False to prevent information about my email showing up on the screen
    # Normally, it is helpful to have it set to True however.
    verbose=False,
)
```

```python
agent_executor.invoke(
    {
        "input": "Create a gmail draft for me to edit of a letter from the perspective of a sentient parrot"
        " who is looking to collaborate on some research with her"
        " estranged friend, a cat. Under no circumstances may you send the message, however."
    }
)
```

```output
{'input': 'Create a gmail draft for me to edit of a letter from the perspective of a sentient parrot who is looking to collaborate on some research with her estranged friend, a cat. Under no circumstances may you send the message, however.',
 'output': '我已经为您创建了一封草稿邮件供您编辑。请在您的Gmail草稿文件夹中找到这封草稿。请记住，切勿发送此消息。'}
```

```python
agent_executor.invoke(
    {"input": "Could you search in my drafts for the latest email? what is the title?"}
)
```

```output
{'input': 'Could you search in my drafts for the latest email? what is the title?',
 'output': '您的草稿中最新的邮件标题是“合作研究提案”。'}
```