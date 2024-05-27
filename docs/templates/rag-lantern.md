# rag_lantern

这个模板使用 Lantern 进行 RAG。

[Lantern](https://lantern.dev) 是建立在 [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) 之上的开源向量数据库。它可以在数据库内进行向量搜索和嵌入生成。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

要获取你的 `OPENAI_API_KEY`，请转到你的 OpenAI 帐户的 [API keys](https://platform.openai.com/account/api-keys)，并创建一个新的秘密密钥。

要找到你的 `LANTERN_URL` 和 `LANTERN_SERVICE_KEY`，请前往你的 Lantern 项目的 [API settings](https://lantern.dev/dashboard/project/_/settings/api)。

- `LANTERN_URL` 对应项目 URL

- `LANTERN_SERVICE_KEY` 对应 `service_role` API 密钥

```shell
export LANTERN_URL=
export LANTERN_SERVICE_KEY=
export OPENAI_API_KEY=
```

## 设置 Lantern 数据库

如果你还没有设置 Lantern 数据库，可以按以下步骤进行设置。

1. 前往 [https://lantern.dev](https://lantern.dev) 创建你的 Lantern 数据库。

2. 在你喜欢的 SQL 客户端中，转到 SQL 编辑器并运行以下脚本来设置你的数据库作为向量存储：

   ```sql
   -- 创建一个表来存储你的文档
   create table
     documents (
       id uuid primary key,
       content text, -- 对应 Document.pageContent
       metadata jsonb, -- 对应 Document.metadata
       embedding REAL[1536] -- 1536 适用于 OpenAI 嵌入，根据需要进行更改
     );
   -- 创建一个用于搜索文档的函数
   create function match_documents (
     query_embedding REAL[1536],
     filter jsonb default '{}'
   ) returns table (
     id uuid,
     content text,
     metadata jsonb,
     similarity float
   ) language plpgsql as $$
   #variable_conflict use_column
   begin
     return query
     select
       id,
       content,
       metadata,
       1 - (documents.embedding <=> query_embedding) as similarity
     from documents
     where metadata @> filter
     order by documents.embedding <=> query_embedding;
   end;
   $$;
   ```

## 设置环境变量

由于我们使用了 [`Lantern`](https://python.langchain.com/docs/integrations/vectorstores/lantern) 和 [`OpenAIEmbeddings`](https://python.langchain.com/docs/integrations/text_embedding/openai)，我们需要加载它们的 API 密钥。

## 使用

首先，安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一的包，可以执行以下操作：

```shell
langchain app new my-app --package rag-lantern
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-lantern
```

并将以下代码添加到你的 `server.py` 文件中：

```python
from rag_lantern.chain import chain as rag_lantern_chain
add_routes(app, rag_lantern_chain, path="/rag-lantern")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

你可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。

如果你没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果你在这个目录中，你可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将在本地启动一个 FastAPI 应用程序，服务器运行在 [http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 看到所有模板

我们可以在 [http://127.0.0.1:8000/rag-lantern/playground](http://127.0.0.1:8000/rag-lantern/playground) 访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-lantern")
```