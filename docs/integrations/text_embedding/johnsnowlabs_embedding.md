# John Snow Labs

[John Snow Labs](https://nlp.johnsnowlabs.com/) 的 NLP & LLM 生态系统包括用于大规模 AI 的软件库、负责任的 AI、无代码 AI，以及超过 20,000 个用于医疗、法律、金融等领域的模型。

模型可以通过 [nlp.load](https://nlp.johnsnowlabs.com/docs/en/jsl/load_api) 进行加载，而 Spark 会话则通过 [nlp.start()](https://nlp.johnsnowlabs.com/docs/en/jsl/start-a-sparksession) 在后台启动。

有关所有 24,000 多个模型，请参阅 [John Snow Labs Model Models Hub](https://nlp.johnsnowlabs.com/models)。

## 设置

```python
%pip install --upgrade --quiet  johnsnowlabs
```

```python
# 如果您有企业许可证，可以运行以下命令安装企业功能
# from johnsnowlabs import nlp
# nlp.install()
```

## 示例

```python
from langchain_community.embeddings.johnsnowlabs import JohnSnowLabsEmbeddings
```

初始化 John Snow Labs Embeddings 和 Spark 会话

```python
embedder = JohnSnowLabsEmbeddings("en.embed_sentence.biobert.clinical_base_cased")
```

定义一些示例文本。这些可以是您想要分析的任何文档，例如新闻文章、社交媒体帖子或产品评论。

```python
texts = ["Cancer is caused by smoking", "Antibiotics aren't painkiller"]
```

生成并打印文本的嵌入。JohnSnowLabsEmbeddings 类为每个文档生成一个嵌入，该嵌入是文档内容的数值表示。这些嵌入可以用于各种自然语言处理任务，例如文档相似性比较或文本分类。

```python
embeddings = embedder.embed_documents(texts)
for i, embedding in enumerate(embeddings):
    print(f"文档 {i+1} 的嵌入：{embedding}")
```

生成并打印单个文本的嵌入。您还可以为单个文本生成嵌入，例如搜索查询。这对于信息检索等任务非常有用，您可以找到与给定查询相似的文档。

```python
query = "Cancer is caused by smoking"
query_embedding = embedder.embed_query(query)
print(f"查询的嵌入：{query_embedding}")
```