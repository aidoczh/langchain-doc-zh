

# nvidia-rag-canonical

这个模板使用 Milvus Vector Store 和 NVIDIA 模型（嵌入和聊天）执行 RAG。

## 环境设置

您应该将您的 NVIDIA API 密钥导出为环境变量。

如果您没有 NVIDIA API 密钥，可以按照以下步骤创建一个：

1. 在 [NVIDIA GPU 云](https://catalog.ngc.nvidia.com/) 服务上创建一个免费帐户，该服务托管 AI 解决方案目录、容器、模型等。

2. 转到 `目录 > AI 基础模型 > (带 API 端点的模型)`。

3. 选择 `API` 选项，然后点击 `生成密钥`。

4. 将生成的密钥保存为 `NVIDIA_API_KEY`。从那里，您应该可以访问端点。

```shell
export NVIDIA_API_KEY=...
```

有关托管 Milvus Vector Store 的说明，请参考底部的部分。

## 用法

要使用这个包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要使用 NVIDIA 模型，安装 Langchain NVIDIA AI Endpoints 包：

```shell
pip install -U langchain_nvidia_aiplay
```

要创建一个新的 LangChain 项目并将其安装为唯一包，您可以执行：

```shell
langchain app new my-app --package nvidia-rag-canonical
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add nvidia-rag-canonical
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from nvidia_rag_canonical import chain as nvidia_rag_canonical_chain
add_routes(app, nvidia_rag_canonical_chain, path="/nvidia-rag-canonical")
```

如果您想设置摄入管道，可以将以下代码添加到您的 `server.py` 文件中：

```python
from nvidia_rag_canonical import ingest as nvidia_rag_ingest
add_routes(app, nvidia_rag_ingest, path="/nvidia-rag-ingest")
```

请注意，通过摄入 API 摄入的文件，需要重新启动服务器才能使新摄入的文件可被检索。

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您还没有要连接的 Milvus Vector Store，请在继续之前查看下面的 `Milvus 设置` 部分。

如果您已经有要连接的 Milvus Vector Store，请编辑 `nvidia_rag_canonical/chain.py` 中的连接详细信息。

如果您在这个目录中，那么您可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 看到所有模板。

我们可以在 [http://127.0.0.1:8000/nvidia-rag-canonical/playground](http://127.0.0.1:8000/nvidia-rag-canonical/playground) 访问 playground。

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/nvidia-rag-canonical")
```

## Milvus 设置

如果您需要创建 Milvus Vector Store 并摄入数据，请使用这个步骤。

我们首先遵循标准的 Milvus 设置说明 [这里](https://milvus.io/docs/install_standalone-docker.md)。

1. 下载 Docker Compose YAML 文件。

    ```shell
    wget https://github.com/milvus-io/milvus/releases/download/v2.3.3/milvus-standalone-docker-compose.yml -O docker-compose.yml
    ```

2. 启动 Milvus Vector Store 容器

    ```shell
    sudo docker compose up -d
    ```

3. 安装 PyMilvus 包以与 Milvus 容器交互。

    ```shell
    pip install pymilvus
    ```

4. 现在让我们摄入一些数据！我们可以通过进入这个目录并运行 `ingest.py` 中的代码来实现，例如：

    ```shell
    python ingest.py
    ```

    请注意，您可以（也应该！）更改此代码以摄入您选择的数据。