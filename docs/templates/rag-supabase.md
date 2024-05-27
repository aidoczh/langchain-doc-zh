

# rag_supabase

这个模板使用 Supabase 进行 RAG。

[Supabase](https://supabase.com/docs) 是一个开源的 Firebase 替代品。它建立在 [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) 之上，后者是一个免费的开源关系数据库管理系统（RDBMS），并使用 [pgvector](https://github.com/pgvector/pgvector) 在您的表中存储嵌入。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

要获取您的 `OPENAI_API_KEY`，请转到您的 OpenAI 帐户的 [API keys](https://platform.openai.com/account/api-keys)，并创建一个新的秘密密钥。

要找到您的 `SUPABASE_URL` 和 `SUPABASE_SERVICE_KEY`，请前往您的 Supabase 项目的 [API settings](https://supabase.com/dashboard/project/_/settings/api)。

- `SUPABASE_URL` 对应项目 URL

- `SUPABASE_SERVICE_KEY` 对应 `service_role` API 密钥

```shell
export SUPABASE_URL=
export SUPABASE_SERVICE_KEY=
export OPENAI_API_KEY=
```

## 设置 Supabase 数据库

如果您还没有设置 Supabase 数据库，请按以下步骤进行设置。

1. 前往 https://database.new 创建您的 Supabase 数据库。

2. 在工作室中，转到 [SQL editor](https://supabase.com/dashboard/project/_/sql/new) 并运行以下脚本以启用 `pgvector` 并将您的数据库设置为向量存储：

   ```sql
   -- 启用 pgvector 扩展以处理嵌入向量
   create extension if not exists vector;
   -- 创建一个表来存储您的文档
   create table
     documents (
       id uuid primary key,
       content text, -- 对应 Document.pageContent
       metadata jsonb, -- 对应 Document.metadata
       embedding vector (1536) -- 1536 适用于 OpenAI 嵌入，根据需要更改
     );
   -- 创建一个用于搜索文档的函数
   create function match_documents (
     query_embedding vector (1536),
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

由于我们使用了 [`SupabaseVectorStore`](https://python.langchain.com/docs/integrations/vectorstores/supabase) 和 [`OpenAIEmbeddings`](https://python.langchain.com/docs/integrations/text_embedding/openai)，我们需要加载它们的 API 密钥。

## 用法

首先，安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一包，可以执行以下操作：

```shell
langchain app new my-app --package rag-supabase
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-supabase
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_supabase.chain import chain as rag_supabase_chain
add_routes(app, rag_supabase_chain, path="/rag-supabase")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

我们可以在 [http://127.0.0.1:8000/rag-supabase/playground](http://127.0.0.1:8000/rag-supabase/playground) 访问 playground

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-supabase")
```

待办事项：添加有关设置 Supabase 数据库的详细信息