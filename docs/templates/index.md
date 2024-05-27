

---

sidebar_class_name: hidden

---

# 模板

突出展示几种不同类别的模板

## ⭐ 热门

以下是一些更受欢迎的模板，供您开始使用。

- [检索增强生成对话机器人](/docs/templates/rag-conversation): 在您的数据上构建对话机器人。默认使用 OpenAI 和 PineconeVectorStore。

- [使用 OpenAI 函数进行提取](/docs/templates/extraction-openai-functions): 从非结构化数据中提取结构化数据。使用 OpenAI 函数调用。

- [本地检索增强生成](/docs/templates/rag-chroma-private): 在您的数据上构建对话机器人。仅使用本地工具：Ollama、GPT4all、Chroma。

- [OpenAI 函数代理](/docs/templates/openai-functions-agent): 构建可以执行操作的对话机器人。使用 OpenAI 函数调用和 Tavily。

- [XML 代理](/docs/templates/xml-agent): 构建可以执行操作的对话机器人。使用 Anthropic 和 You.com。

## 📥 高级检索

这些模板涵盖了高级检索技术，可用于在数据库或文档上进行聊天和问答。

- [重新排序](/docs/templates/rag-pinecone-rerank): 此检索技术使用 Cohere 的重新排序端点来重新排序从初始检索步骤中检索到的文档。

- [Anthropic 迭代搜索](/docs/templates/anthropic-iterative-search): 此检索技术使用迭代提示来确定要检索的内容以及检索到的文档是否足够好。

- 使用 [Neo4j](/docs/templates/neo4j-parent) 或 [MongoDB](/docs/templates/mongo-parent-document-retrieval) 进行**父文档检索**：此检索技术存储较小块的嵌入，然后返回较大块以传递给模型进行生成。

- [半结构化 RAG](/docs/templates/rag-semi-structured): 该模板展示了如何在半结构化数据（例如涉及文本和表格的数据）上进行检索。

- [时间 RAG](/docs/templates/rag-timescale-hybrid-search-time): 该模板展示了如何使用带有基于时间组件的数据的混合搜索进行检索，使用 [Timescale Vector](https://www.timescale.com/ai?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral)。

## 🔍高级检索 - 查询转换

一些涉及转换原始用户查询的高级检索方法，可以提高检索质量。

- [假设文档嵌入](/docs/templates/hyde): 一种检索技术，为给定查询生成假设文档，然后使用该文档的嵌入进行语义搜索。[论文](https://arxiv.org/abs/2212.10496)。

- [重写-检索-阅读](/docs/templates/rewrite-retrieve-read): 一种在将查询传递给搜索引擎之前重写给定查询的检索技术。[论文](https://arxiv.org/abs/2305.14283)。

- [回退问答提示](/docs/templates/stepback-qa-prompting): 一种生成“回退”问题，然后检索与该问题和原始问题都相关的文档的检索技术。[论文](https://arxiv.org/abs//2310.06117)。

- [RAG 融合](/docs/templates/rag-fusion): 一种生成多个查询，然后使用互惠排名融合重新排列检索到的文档的检索技术。[文章](https://towardsdatascience.com/forget-rag-the-future-is-rag-fusion-1147298d8ad1)。

- [多查询检索器](/docs/templates/rag-pinecone-multi-query): 此检索技术使用 LLM 生成多个查询，然后为所有查询获取文档。

## 🧠高级检索 - 查询构建

一些涉及在自然语言之外的单独 DSL 中构建查询的高级检索方法，可实现对各种结构化数据库的自然语言聊天。

- [弹性查询生成器](/docs/templates/elastic-query-generator): 从自然语言生成弹性搜索查询。

- [Neo4j Cypher 生成](/docs/templates/neo4j-cypher): 从自然语言生成 Cypher 语句。还提供一个带有 ["全文" 选项](/docs/templates/neo4j-cypher-ft)。

- [Supabase 自查询](/docs/templates/self-query-supabase): 将自然语言查询解析为语义查询以及 Supabase 的元数据过滤器。

## 🦙 开源模型

这些模板使用开源模型，可为敏感数据提供隐私保护。

- [本地检索增强生成](/docs/templates/rag-chroma-private): 在您的数据上构建对话机器人。仅使用本地工具：Ollama、GPT4all、Chroma。

- [SQL 问答（复制）](/docs/templates/sql-llama2): 在 SQL 数据库上进行问答，使用托管在 [Replicate](https://replicate.com/) 上的 Llama2。

- [SQL 问答（LlamaCpp）](/docs/templates/sql-llamacpp): 在 SQL 数据库上进行问答，通过 [LlamaCpp](https://github.com/ggerganov/llama.cpp) 使用 Llama2。

- [SQL问题回答（Ollama）](/docs/templates/sql-ollama)：在SQL数据库上进行问题回答，使用Llama2通过[Ollama](https://github.com/jmorganca/ollama)实现。

## ⛏️ 数据提取

这些模板根据用户指定的模式以结构化格式提取数据。

- [使用OpenAI函数进行提取](/docs/templates/extraction-openai-functions)：使用OpenAI函数调用从文本中提取信息。

- [使用Anthropic函数进行提取](/docs/templates/extraction-anthropic-functions)：使用围绕Anthropic端点的LangChain包装器从文本中提取信息，旨在模拟函数调用。

- [提取生物技术板数据](/docs/templates/plate-chain)：从混乱的Excel电子表格中提取微孔板数据，转换为更规范化的格式。

## ⛏️ 摘要和标记

这些模板摘要或对文档和文本进行分类。

- [使用Anthropic进行摘要](/docs/templates/summarize-anthropic)：使用Anthropic的Claude2对长文档进行摘要。

## 🤖 代理

这些模板构建可以执行操作的聊天机器人，帮助自动化任务。

- [OpenAI函数代理](/docs/templates/openai-functions-agent)：构建可以执行操作的聊天机器人。使用OpenAI函数调用和Tavily。

- [XML代理](/docs/templates/xml-agent)：构建可以执行操作的聊天机器人。使用Anthropic和You.com。

## :rotating_light: 安全和评估

这些模板使LLM输出的调节或评估成为可能。

- [Guardrails输出解析器](/docs/templates/guardrails-output-parser)：使用guardrails-ai验证LLM输出。

- [聊天机器人反馈](/docs/templates/chat-bot-feedback)：使用LangSmith评估聊天机器人的回应。