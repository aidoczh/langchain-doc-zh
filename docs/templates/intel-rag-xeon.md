# 在 Intel Xeon 上的 RAG 示例

该模板使用 Chroma 和文本生成推理在 Intel® Xeon® 可扩展处理器上执行 RAG。

Intel® Xeon® 可扩展处理器具有内置加速器，可实现更高的每核性能和无与伦比的人工智能性能，配备先进的安全技术，满足最高需求的工作负载要求，同时提供最大的云选择和应用可移植性，请查看[Intel® Xeon® 可扩展处理器](https://www.intel.com/content/www/us/en/products/details/processors/xeon/scalable.html)。

## 环境设置

要在 Intel® Xeon® 可扩展处理器上使用[🤗 文本生成推理](https://github.com/huggingface/text-generation-inference)，请按照以下步骤操作：

### 在 Intel Xeon 服务器上启动本地服务器实例：

```bash
model=Intel/neural-chat-7b-v3-3
volume=$PWD/data # 与 Docker 容器共享卷，避免每次运行都下载权重
docker run --shm-size 1g -p 8080:80 -v $volume:/data ghcr.io/huggingface/text-generation-inference:1.4 --model-id $model
```

对于像 `LLAMA-2` 这样的门控模型，您需要在上述 docker 运行命令中传递 -e HUGGING_FACE_HUB_TOKEN=\<token\>，并使用有效的 Hugging Face Hub 读取令牌。

请按照此链接[huggingface token](https://huggingface.co/docs/hub/security-tokens)获取访问令牌，并使用该令牌导出 `HUGGINGFACEHUB_API_TOKEN` 环境变量。

```bash
export HUGGINGFACEHUB_API_TOKEN=<token> 
```

发送请求以检查端点是否正常工作：

```bash
curl localhost:8080/generate -X POST -d '{"inputs":"Which NFL team won the Super Bowl in the 2010 season?","parameters":{"max_new_tokens":128, "do_sample": true}}'   -H 'Content-Type: application/json'
```

更多细节请参考[text-generation-inference](https://github.com/huggingface/text-generation-inference)。

## 数据填充

如果您想要使用一些示例数据填充数据库，可以运行以下命令：

```shell
poetry install
poetry run python ingest.py
```

该脚本会处理并将来自 Nike `nke-10k-2023.pdf` 的 Edgar 10k 文件数据部分存储到 Chroma 数据库中。

## 使用

要使用此软件包，您首先应安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package intel-rag-xeon
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add intel-rag-xeon
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from intel_rag_xeon import chain as xeon_rag_chain
add_routes(app, xeon_rag_chain, path="/intel-rag-xeon")
```

（可选）现在让我们配置 LangSmith。LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。您可以在此处注册 LangSmith [here](https://smith.langchain.com/)。如果您没有访问权限，可以跳过此部分

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

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/intel-rag-xeon/playground](http://127.0.0.1:8000/intel-rag-xeon/playground)访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/intel-rag-xeon")
```