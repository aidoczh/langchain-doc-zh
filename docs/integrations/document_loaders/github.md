# GitHub

这个笔记本展示了如何在 [GitHub](https://github.com/) 上加载给定存储库的问题和拉取请求（PRs）。还展示了如何在 [GitHub](https://github.com/) 上为给定存储库加载 github 文件。我们将使用 LangChain Python 存储库作为示例。

## 设置访问令牌

要访问 GitHub API，您需要一个个人访问令牌 - 您可以在这里设置您的: https://github.com/settings/tokens?type=beta。您可以将此令牌设置为环境变量 ``GITHUB_PERSONAL_ACCESS_TOKEN``，它将被自动引入，或者您可以直接在初始化时将其作为 ``access_token`` 命名参数传递进去。

```python
# 如果您还没有将访问令牌设置为环境变量，请在这里传递进来。
from getpass import getpass
ACCESS_TOKEN = getpass()
```

## 加载问题和 PRs

```python
from langchain_community.document_loaders import GitHubIssuesLoader
```

```python
loader = GitHubIssuesLoader(
    repo="langchain-ai/langchain",
    access_token=ACCESS_TOKEN,  # 如果您已经将访问令牌设置为环境变量，请删除/注释掉这个参数。
    creator="UmerHA",
)
```

让我们加载所有由 "UmerHA" 创建的问题和 PRs。

以下是您可以使用的所有过滤器列表:

- include_prs

- milestone

- state

- assignee

- creator

- mentioned

- labels

- sort

- direction

- since

更多信息，请参阅 https://docs.github.com/en/rest/issues/issues?apiVersion=2022-11-28#list-repository-issues。

```python
docs = loader.load()
```

```python
print(docs[0].page_content)
print(docs[0].metadata)
```

## 仅加载问题

默认情况下，GitHub API 返回的内容中也包括拉取请求。要仅获取 '纯' 问题（即，不包括拉取请求），请使用 `include_prs=False`。

```python
loader = GitHubIssuesLoader(
    repo="langchain-ai/langchain",
    access_token=ACCESS_TOKEN,  # 如果您已经将访问令牌设置为环境变量，请删除/注释掉这个参数。
    creator="UmerHA",
    include_prs=False,
)
docs = loader.load()
```

```python
print(docs[0].page_content)
print(docs[0].metadata)
```

## 加载 Github 文件内容

对于下面的代码，加载存储库 `langchain-ai/langchain` 中的所有 markdown 文件。

```python
from langchain.document_loaders import GithubFileLoader
```

```python
loader = GithubFileLoader(
    repo="langchain-ai/langchain",  # 存储库名称
    access_token=ACCESS_TOKEN,
    github_api_url="https://api.github.com",
    file_filter=lambda file_path: file_path.endswith(
        ".md"
    ),  # 加载所有 markdown 文件。
)
documents = loader.load()
```

一个文档的示例输出:

```json
documents.metadata: 
    {
      "path": "README.md",
      "sha": "82f1c4ea88ecf8d2dfsfx06a700e84be4",
      "source": "https://github.com/langchain-ai/langchain/blob/master/README.md"
    }
documents.content:
    模拟内容
```