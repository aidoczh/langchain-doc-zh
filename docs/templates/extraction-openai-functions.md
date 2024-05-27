# extraction-openai-functions

这个模板使用[OpenAI函数调用](https://python.langchain.com/docs/modules/chains/how_to/openai_functions)来从非结构化输入文本中提取结构化输出。

提取的输出模式可以在`chain.py`中设置。

## 环境设置

设置`OPENAI_API_KEY`环境变量以访问OpenAI模型。

## 使用方法

要使用这个包，你首先需要安装LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的LangChain项目并将其安装为唯一的包，可以执行以下操作：

```shell
langchain app new my-app --package extraction-openai-functions
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add extraction-openai-functions
```

并将以下代码添加到你的`server.py`文件中：

```python
from extraction_openai_functions import chain as extraction_openai_functions_chain
add_routes(app, extraction_openai_functions_chain, path="/extraction-openai-functions")
```

（可选）现在让我们配置LangSmith。

LangSmith将帮助我们跟踪、监控和调试LangChain应用程序。

你可以在[这里](https://smith.langchain.com/)注册LangSmith。

如果你没有访问权限，你可以跳过这一部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果你在这个目录下，你可以直接启动一个LangServe实例：

```shell
langchain serve
```

这将启动一个FastAPI应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)上看到所有模板。

我们可以在[http://127.0.0.1:8000/extraction-openai-functions/playground](http://127.0.0.1:8000/extraction-openai-functions/playground)上访问playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/extraction-openai-functions")
```

默认情况下，该包设置为提取论文的标题和作者，如`chain.py`文件中所指定。

OpenAI函数默认使用LLM。