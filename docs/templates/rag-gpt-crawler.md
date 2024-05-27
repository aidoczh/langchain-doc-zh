

# rag-gpt-crawler

GPT-crawler 会爬取网站以生成用于自定义 GPT 或其他应用程序（RAG）的文件。

此模板使用 [gpt-crawler](https://github.com/BuilderIO/gpt-crawler) 构建 RAG 应用程序。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 爬取

运行 GPT-crawler 从一组 URL 中提取内容，使用 GPT-crawler 仓库中的配置文件。

以下是 LangChain 用例文档的示例配置：

```
export const config: Config = {
  url: "https://python.langchain.com/docs/use_cases/",
  match: "https://python.langchain.com/docs/use_cases/**",
  selector: ".docMainContainer_gTbr",
  maxPagesToCrawl: 10,
  outputFileName: "output.json",
};
```

然后，按照 [gpt-crawler](https://github.com/BuilderIO/gpt-crawler) 的 README 中的说明运行：

```
npm start
```

并将 `output.json` 文件复制到包含此 README 的文件夹中。

## 使用方法

要使用此软件包，您应首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，可以执行以下操作：

```shell
langchain app new my-app --package rag-gpt-crawler
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-gpt-crawler
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_chroma import chain as rag_gpt_crawler
add_routes(app, rag_gpt_crawler, path="/rag-gpt-crawler")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在 [此处](https://smith.langchain.com/) 注册 LangSmith。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们可以在 [http://127.0.0.1:8000/rag-gpt-crawler/playground](http://127.0.0.1:8000/rag-gpt-crawler/playground) 访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-gpt-crawler")
```