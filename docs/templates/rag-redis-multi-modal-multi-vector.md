

# rag-redis-multi-modal-multi-vector

多模态LLM使得视觉助手能够对图像进行问答。

该模板创建了一个用于幻灯片演示的视觉助手，通常包含图表或图片等视觉元素。

它使用GPT-4V为每张幻灯片创建图像摘要，将摘要嵌入并存储在Redis中。

给定一个问题，相关幻灯片将被检索并传递给GPT-4V进行答案合成。

## 输入

在`/docs`目录中提供一个PDF格式的幻灯片演示文稿。

默认情况下，此模板包含有关NVIDIA最近收益的幻灯片演示文稿。

可以提出的示例问题包括：

```
1/ H100 TensorRT可以提高LLama2推理性能多少？
2/ GPU加速应用程序从2020年到2023年的变化率是多少？
```

要创建幻灯片演示文稿的索引，请运行：

```
poetry install
poetry shell
python ingest.py
```

## 存储

以下是模板将使用的过程来创建幻灯片索引（参见[博客](https://blog.langchain.dev/multi-modal-rag-template/)）：

* 将幻灯片提取为图像集合

* 使用GPT-4V对每个图像进行摘要

* 使用文本嵌入嵌入图像摘要，并附带原始图像的链接

* 基于图像摘要与用户输入问题之间的相似性检索相关图像

* 将这些图像传递给GPT-4V进行答案合成

### Redis

此模板使用[Redis](https://redis.com)来支持[MultiVectorRetriever](https://python.langchain.com/docs/modules/data_connection/retrievers/multi_vector)，包括：

- Redis作为[VectorStore](https://python.langchain.com/docs/integrations/vectorstores/redis)（用于存储+索引图像摘要嵌入）

- Redis作为[ByteStore](https://python.langchain.com/docs/integrations/stores/redis)（用于存储图像）

请确保部署一个Redis实例，可以在[云端](https://redis.com/try-free)（免费）或本地使用[docker](https://redis.io/docs/install/install-stack/docker/)。

这将为您提供一个可访问的Redis端点，您可以将其用作URL。如果在本地部署，请简单地使用`redis://localhost:6379`。

## LLM

该应用程序将根据文本输入与图像摘要（文本）之间的相似性检索图像，并将图像传递给GPT-4V进行答案合成。

## 环境设置

设置`OPENAI_API_KEY`环境变量以访问OpenAI GPT-4V。

设置`REDIS_URL`环境变量以访问您的Redis数据库。

## 使用

要使用此软件包，您首先应安装LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的LangChain项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package rag-redis-multi-modal-multi-vector
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-redis-multi-modal-multi-vector
```

并将以下代码添加到您的`server.py`文件中：

```python
from rag_redis_multi_modal_multi_vector import chain as rag_redis_multi_modal_chain_mv
add_routes(app, rag_redis_multi_modal_chain_mv, path="/rag-redis-multi-modal-multi-vector")
```

（可选）现在让我们配置LangSmith。

LangSmith将帮助我们跟踪、监视和调试LangChain应用程序。

您可以在[此处](https://smith.langchain.com/)注册LangSmith。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果您在此目录中，则可以通过以下方式直接启动LangServe实例：

```shell
langchain serve
```

这将在本地启动一个运行的FastAPI应用程序，服务器位于

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)查看所有模板。

我们可以在[http://127.0.0.1:8000/rag-redis-multi-modal-multi-vector/playground](http://127.0.0.1:8000/rag-redis-multi-modal-multi-vector/playground)访问playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-redis-multi-modal-multi-vector")
```