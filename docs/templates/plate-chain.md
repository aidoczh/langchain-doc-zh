# 板链

该模板能够解析实验板上的数据。

在生物化学或分子生物学的背景下，实验板是常用的工具，用于以网格形式保存样本。

这可以将生成的数据解析为标准化的（例如 JSON）格式，以便进行进一步处理。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 用法

要使用板链，您必须安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

创建一个新的 LangChain 项目并将板链安装为唯一的包可以通过以下方式完成：

```shell
langchain app new my-app --package plate-chain
```

如果您希望将其添加到现有项目中，只需运行：

```shell
langchain app add plate-chain
```

然后将以下代码添加到您的 `server.py` 文件中：

```python
from plate_chain import chain as plate_chain
add_routes(app, plate_chain, path="/plate-chain")
```

（可选）要配置 LangSmith，以帮助跟踪、监视和调试 LangChain 应用程序，请使用以下代码：

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动一个在本地运行的服务器的 FastAPI 应用程序，地址为

[http://localhost:8000](http://localhost:8000)

所有模板都可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看

在 [http://127.0.0.1:8000/plate-chain/playground](http://127.0.0.1:8000/plate-chain/playground) 访问 playground  

您可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/plate-chain")
```