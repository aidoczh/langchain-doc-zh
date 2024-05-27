

# SQL-llamacpp

这个模板使用户能够使用自然语言与 SQL 数据库进行交互。

它使用 [Mistral-7b](https://mistral.ai/news/announcing-mistral-7b/) 通过 [llama.cpp](https://github.com/ggerganov/llama.cpp) 在 Mac 笔记本电脑上本地运行推理。

## 环境设置

要设置环境，请按照以下步骤操作：

```shell
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh
bash Miniforge3-MacOSX-arm64.sh
conda create -n llama python=3.9.16
conda activate /Users/rlm/miniforge3/envs/llama
CMAKE_ARGS="-DLLAMA_METAL=on" FORCE_CMAKE=1 pip install -U llama-cpp-python --no-cache-dir
```

## 用法

要使用这个包，您应该首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一包，您可以执行：

```shell
langchain app new my-app --package sql-llamacpp
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add sql-llamacpp
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from sql_llamacpp import chain as sql_llamacpp_chain
add_routes(app, sql_llamacpp_chain, path="/sql-llamacpp")
```

该包将从 [这里](https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.1-GGUF) 下载 Mistral-7b 模型。您可以选择其他文件并指定它们的下载路径（浏览 [这里](https://huggingface.co/TheBloke)）。

这个包包含一个包含 2023 年 NBA 球员名单的示例数据库。您可以查看构建此数据库的说明 [这里](https://github.com/facebookresearch/llama-recipes/blob/main/demo_apps/StructuredLlama.ipynb)。

（可选）为了跟踪、监控和调试 LangChain 应用程序，您可以配置 LangSmith。您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动一个在本地运行的服务器的 FastAPI 应用程序，地址为

[http://localhost:8000](http://localhost:8000)

您可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

您可以在 [http://127.0.0.1:8000/sql-llamacpp/playground](http://127.0.0.1:8000/sql-llamacpp/playground) 访问 playground

您可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/sql-llamacpp")
```