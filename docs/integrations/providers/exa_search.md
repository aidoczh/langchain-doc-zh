# Exa 搜索

Exa 的搜索集成存在于其独立的[合作伙伴包](https://pypi.org/project/langchain-exa/)中。您可以使用以下命令进行安装：

```python
%pip install -qU langchain-exa
```

为了使用该包，您还需要将 `EXA_API_KEY` 环境变量设置为您的 Exa API 密钥。

## 检索器

您可以在标准检索流程中使用 [`ExaSearchRetriever`](/docs/integrations/tools/exa_search#using-exasearchretriever)。您可以按以下方式导入它：

```python
from langchain_exa import ExaSearchRetriever
```

## 工具

您可以像在[Exa工具调用文档](/docs/integrations/tools/exa_search#using-the-exa-sdk-as-langchain-agent-tools)中描述的那样，将 Exa 作为代理工具使用。