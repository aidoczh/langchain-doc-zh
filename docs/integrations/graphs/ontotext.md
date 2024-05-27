# Ontotext GraphDB

[Ontotext GraphDB](https://graphdb.ontotext.com/) 是一个符合[RDF](https://www.w3.org/RDF/)和[SPARQL](https://www.w3.org/TR/sparql11-query/)标准的图数据库和知识发现工具。

本文档展示了如何使用LLMs提供自然语言查询（NLQ到SPARQL，也称为`text2sparql`）来查询`Ontotext GraphDB`。

## GraphDB LLM功能

`GraphDB`支持一些LLM集成功能，如[这里](https://github.com/w3c/sparql-dev/issues/193)所述：

[gpt-queries](https://graphdb.ontotext.com/documentation/10.5/gpt-queries.html)

- 使用知识图（KG）中的数据，通过魔术谓词向LLM询问文本、列表或表

- 查询解释

- 结果解释、总结、改写、翻译

[retrieval-graphdb-connector](https://graphdb.ontotext.com/documentation/10.5/retrieval-graphdb-connector.html)

- 在向量数据库中索引KG实体

- 支持任何文本嵌入算法和向量数据库

- 使用与GraphDB用于Elastic、Solr、Lucene相同的强大连接器（索引）语言

- 自动同步RDF数据对KG实体索引的更改

- 支持嵌套对象（GraphDB版本10.5中没有UI支持）

- 将KG实体序列化为文本，例如（用于葡萄酒数据集）：

```
Franvino:
- 是红葡萄酒。
- 由梅洛葡萄酿制。
- 由品丽珠葡萄酿制。
- 干燥的糖分。
- 年份为2012年。
```

[talk-to-graph](https://graphdb.ontotext.com/documentation/10.5/talk-to-graph.html)

- 使用定义的KG实体索引的简单聊天机器人

在本教程中，我们不会使用GraphDB LLM集成，而是从NLQ生成`SPARQL`。我们将使用`Star Wars API`（`SWAPI`）本体和数据集，您可以在[这里](https://github.com/Ontotext-AD/langchain-graphdb-qa-chain-demo/blob/main/starwars-data.trig)查看。

## 设置

您需要一个正在运行的GraphDB实例。本教程展示了如何使用[GraphDB Docker镜像](https://hub.docker.com/r/ontotext/graphdb)在本地运行数据库。它提供了一个docker compose设置，该设置会使用Star Wars数据集填充GraphDB。所有必要的文件，包括本笔记本，都可以从[GitHub存储库langchain-graphdb-qa-chain-demo](https://github.com/Ontotext-AD/langchain-graphdb-qa-chain-demo)下载。

* 安装[Docker](https://docs.docker.com/get-docker/)。本教程是使用捆绑了[Docker Compose](https://docs.docker.com/compose/)的Docker版本`24.0.7`创建的。对于较早的Docker版本，您可能需要单独安装Docker Compose。

* 在计算机的本地文件夹中克隆[GitHub存储库langchain-graphdb-qa-chain-demo](https://github.com/Ontotext-AD/langchain-graphdb-qa-chain-demo)。

* 使用以下脚本从相同文件夹执行，启动GraphDB

```
docker build --tag graphdb .
docker compose up -d graphdb
```

您需要等待几秒钟，数据库才会在`http://localhost:7200/`上启动。Star Wars数据集`starwars-data.trig`会自动加载到`langchain`存储库中。本地SPARQL端点`http://localhost:7200/repositories/langchain`可用于运行查询。您还可以从您喜欢的Web浏览器打开GraphDB Workbench `http://localhost:7200/sparql`，在那里可以进行交互式查询。

* 设置工作环境

如果您使用`conda`，请创建并激活一个新的conda环境（例如`conda create -n graph_ontotext_graphdb_qa python=3.9.18`）。

安装以下库：

```
pip install jupyter==1.0.0
pip install openai==1.6.1
pip install rdflib==7.0.0
pip install langchain-openai==0.0.2
pip install langchain>=0.1.5
```

使用以下命令运行Jupyter：

```
jupyter notebook
```

## 指定本体

为了使LLM能够生成SPARQL，它需要知识图模式（本体）。可以使用`OntotextGraphDBGraph`类的两个参数之一来提供：

* `query_ontology`：在SPARQL端点上执行的`CONSTRUCT`查询，返回KG模式语句。我们建议将本体存储在自己的命名图中，这样可以更容易地获取相关语句（如下例）。不支持`DESCRIBE`查询，因为`DESCRIBE`返回对称简洁边界描述（SCBD），即还包括传入的类链接。对于具有数百万实例的大图，这是低效的。请参阅https://github.com/eclipse-rdf4j/rdf4j/issues/4857

* `local_file`：本地RDF本体文件。支持的RDF格式包括`Turtle`、`RDF/XML`、`JSON-LD`、`N-Triples`、`Notation-3`、`Trig`、`Trix`、`N-Quads`。

无论哪种情况，本体转储应包括：

* 足够的关于类、属性、属性附加到类（使用rdfs:domain、schema:domainIncludes或OWL限制）和分类法（重要个体）的信息。

* 不包括过于冗长和与 SPARQL 构建无关的定义和示例。

```python
from langchain_community.graphs import OntotextGraphDBGraph
# 使用用户构造查询来提供模式
graph = OntotextGraphDBGraph(
    query_endpoint="http://localhost:7200/repositories/langchain",
    query_ontology="CONSTRUCT {?s ?p ?o} FROM <https://swapi.co/ontology/> WHERE {?s ?p ?o}",
)
```

```python
# 使用本地 RDF 文件来提供模式
graph = OntotextGraphDBGraph(
    query_endpoint="http://localhost:7200/repositories/langchain",
    local_file="/path/to/langchain_graphdb_tutorial/starwars-ontology.nt",  # 在这里更改路径
)
```

无论哪种方式，本体（模式）都以 `Turtle` 形式提供给 LLM，因为带有适当前缀的 `Turtle` 最紧凑且最容易让 LLM 记住。

星球大战本体有点不寻常，因为它包含了许多关于类的具体三元组，例如物种 `:Aleena` 生活在 `<planet/38>` 上，它们是 `:Reptile` 的子类，具有某些典型特征（平均身高、平均寿命、皮肤颜色），以及特定个体（角色）是该类的代表：

```turtle
@prefix : <https://swapi.co/vocabulary/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
:Aleena a owl:Class, :Species ;
    rdfs:label "Aleena" ;
    rdfs:isDefinedBy <https://swapi.co/ontology/> ;
    rdfs:subClassOf :Reptile, :Sentient ;
    :averageHeight 80.0 ;
    :averageLifespan "79" ;
    :character <https://swapi.co/resource/aleena/47> ;
    :film <https://swapi.co/resource/film/4> ;
    :language "Aleena" ;
    :planet <https://swapi.co/resource/planet/38> ;
    :skinColor "blue", "gray" .
    ...
```

为了使本教程简单，我们使用了未加密的 GraphDB。如果 GraphDB 是受保护的，您应该在初始化 `OntotextGraphDBGraph` 之前设置环境变量 'GRAPHDB_USERNAME' 和 'GRAPHDB_PASSWORD'。

```python
import os
os.environ["GRAPHDB_USERNAME"] = "graphdb-user"
os.environ["GRAPHDB_PASSWORD"] = "graphdb-password"
graph = OntotextGraphDBGraph(
    query_endpoint=...,
    query_ontology=...
)
```

## 对 StarWars 数据集进行问答

现在我们可以使用 `OntotextGraphDBQAChain` 来提问。

```python
import os
from langchain.chains import OntotextGraphDBQAChain
from langchain_openai import ChatOpenAI
# 我们将使用一个需要 OpenAI API 密钥的 OpenAI 模型。
# 但是也有其他可用的模型：
# https://python.langchain.com/docs/integrations/chat/
# 将环境变量 `OPENAI_API_KEY` 设置为您的 OpenAI API 密钥
os.environ["OPENAI_API_KEY"] = "sk-***"
# 这里可以使用任何可用的 OpenAI 模型。
# 我们使用 'gpt-4-1106-preview' 是因为它有更大的上下文窗口。
# 'gpt-4-1106-preview' 模型名称将来会被弃用，并更改为 'gpt-4-turbo' 或类似的，
# 因此请务必参考 OpenAI API https://platform.openai.com/docs/models 获取正确的命名。
chain = OntotextGraphDBQAChain.from_llm(
    ChatOpenAI(temperature=0, model_name="gpt-4-1106-preview"),
    graph=graph,
    verbose=True,
)
```

让我们问一个简单的问题。

```python
chain.invoke({chain.input_key: "Tatooine 的气候是什么？"})[chain.output_key]
```

```output
> 进入新的 OntotextGraphDBQAChain 链...
生成的 SPARQL：
PREFIX : <https://swapi.co/vocabulary/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?climate
WHERE {
  ?planet rdfs:label "Tatooine" ;
          :climate ?climate .
}
> 链完成。
```

```output
'Tatooine 的气候是干旱的。'
```

再问一个稍微复杂一点的问题。

```python
chain.invoke({chain.input_key: "Luke Skywalker 的家乡的气候是什么？"})[chain.output_key]
```

```output
> 进入新的 OntotextGraphDBQAChain 链...
生成的 SPARQL：
PREFIX : <https://swapi.co/vocabulary/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?climate
WHERE {
  ?character rdfs:label "Luke Skywalker" .
  ?character :homeworld ?planet .
  ?planet :climate ?climate .
}
> 链完成。
```

```output
"Luke Skywalker 的家乡的气候是干旱的。"
```

我们还可以问一些更复杂的问题，比如

```python
chain.invoke(
    {
        chain.input_key: "所有星球大战电影的平均票房收入是多少？"
    }
)[chain.output_key]
```

```output
> 进入新的 OntotextGraphDBQAChain 链...
生成的 SPARQL：
PREFIX : <https://swapi.co/vocabulary/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT (AVG(?boxOffice) AS ?averageBoxOffice)
WHERE {
  ?film a :Film .
  ?film :boxOffice ?boxOfficeValue .
  BIND(xsd:decimal(?boxOfficeValue) AS ?boxOffice)
}
> 链完成。
```

星球大战电影的平均票房收入约为7.541亿美元。

## 链式修改器

Ontotext GraphDB QA 链允许对您的 QA 链进行及时的改进，以增强您的应用的整体用户体验。

### "SPARQL 生成"提示

该提示用于基于用户问题和知识图谱模式生成 SPARQL 查询。

- `sparql_generation_prompt`

  默认值:

  ```python
  GRAPHDB_SPARQL_GENERATION_TEMPLATE = """
  编写一个用于查询图数据库的 SPARQL SELECT 查询。
  本体模式以 Turtle 格式的三个反引号分隔如下：
  ```

  {schema}

  ```
  仅使用模式中提供的类和属性构建 SPARQL 查询。
  不要使用在 SPARQL 查询中未明确提供的任何类或属性。
  包括所有必要的前缀。
  不要在响应中包含任何解释或道歉。
  不要用反引号包裹查询。
  不要在响应中包含除生成的 SPARQL 查询之外的任何文本。
  问题以三个反引号分隔如下：
  ```

  {prompt}

  ```
  """
  GRAPHDB_SPARQL_GENERATION_PROMPT = PromptTemplate(
      input_variables=["schema", "prompt"],
      template=GRAPHDB_SPARQL_GENERATION_TEMPLATE,
  )
  ```

### "SPARQL 修复"提示

有时，LLM 可能会生成带有语法错误或缺少前缀等的 SPARQL 查询。该链将尝试通过提示LLM在一定次数内进行更正。

- `sparql_fix_prompt`

  默认值:

  ```python
  GRAPHDB_SPARQL_FIX_TEMPLATE = """
  以下由三个反引号分隔的 SPARQL 查询
  ```

  {generated_sparql}

  ```
  是无效的。
  由三个反引号分隔的错误如下
  ```

  {error_message}

  ```
  给我一个正确版本的 SPARQL 查询。
  不要改变查询的逻辑。
  不要在响应中包含任何解释或道歉。
  不要用反引号包裹查询。
  不要在响应中包含除生成的 SPARQL 查询之外的任何文本。
  本体模式以 Turtle 格式的三个反引号分隔如下：
  ```

  {schema}

  ```
  """
  GRAPHDB_SPARQL_FIX_PROMPT = PromptTemplate(
      input_variables=["error_message", "generated_sparql", "schema"],
      template=GRAPHDB_SPARQL_FIX_TEMPLATE,
  )
  ```

- `max_fix_retries`

  默认值: `5`

### "回答"提示

该提示用于根据从数据库返回的结果和初始用户问题来回答问题。默认情况下，LLM 被指示仅使用返回结果中的信息。如果结果集为空，LLM 应通知无法回答问题。

- `qa_prompt`

  默认值:

  ```python
  GRAPHDB_QA_TEMPLATE = """任务：从 SPARQL 查询的结果生成自然语言响应。
  您是一个创建写作精良且人类可理解的答案的助手。
  信息部分包含提供的信息，您可以使用它来构建答案。
  提供的信息是权威的，您绝不能怀疑它或尝试使用内部知识来更正它。
  使您的响应听起来像是来自 AI 助手的信息，但不要添加任何信息。
  不要使用内部知识来回答问题，如果没有可用的信息，只需说您不知道。
  信息：
  {context}
  问题：{prompt}
  有用的答案:"""
  GRAPHDB_QA_PROMPT = PromptTemplate(
      input_variables=["context", "prompt"], template=GRAPHDB_QA_TEMPLATE
  )
  ```

完成与 GraphDB 的 QA 玩耍后，您可以通过在具有 Docker Compose 文件的目录中运行以下命令来关闭 Docker 环境

```
docker compose down -v --remove-orphans
```