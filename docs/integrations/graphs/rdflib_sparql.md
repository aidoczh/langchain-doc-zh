# RDFLib

[RDFLib](https://rdflib.readthedocs.io/) 是一个纯 Python 包，用于处理[RDF](https://en.wikipedia.org/wiki/Resource_Description_Framework)。`RDFLib`包含了处理`RDF`所需的大部分功能，包括：

- 用于 RDF/XML、N3、NTriples、N-Quads、Turtle、TriX、Trig 和 JSON-LD 的解析器和序列化器

- 一个图形接口，可以由多种存储实现支持

- 用于内存、磁盘持久化（Berkeley DB）和远程 SPARQL 端点的存储实现

- 一个支持 SPARQL 1.1 的实现 - 支持 SPARQL 1.1 查询和更新语句

- SPARQL 函数扩展机制

图数据库是基于类似网络模型的应用程序的绝佳选择。为了标准化这些图的语法和语义，W3C推荐使用`语义网技术`，参见[语义网](https://www.w3.org/standards/semanticweb/)。

[SPARQL](https://www.w3.org/TR/sparql11-query/)作为查询语言类似于`SQL`或`Cypher`用于这些图。本笔记本演示了将LLMs应用作为自然语言接口到图数据库的过程，通过生成`SPARQL`。

**免责声明：**到目前为止，通过LLMs生成`SPARQL`查询仍然有些不稳定。特别要小心`UPDATE`查询，因为它们会改变图形。

## 设置

我们需要安装一个 Python 库：

```python
!pip install rdflib
```

有几个数据源可以运行查询，包括网络上的文件、本地可用的文件、SPARQL 端点，例如[维基数据](https://www.wikidata.org/wiki/Wikidata:Main_Page)，以及[三元组存储](https://www.w3.org/wiki/LargeTripleStores)。

```python
from langchain.chains import GraphSparqlQAChain
from langchain_community.graphs import RdfGraph
from langchain_openai import ChatOpenAI
```

```python
graph = RdfGraph(
    source_file="http://www.w3.org/People/Berners-Lee/card",
    standard="rdf",
    local_copy="test.ttl",
)
```

请注意，如果源是只读的，则提供`local_file`是必要的，用于在本地存储更改。

## 刷新图模式信息

如果数据库的模式发生更改，您可以刷新生成 SPARQL 查询所需的模式信息。

```python
graph.load_schema()
```

```python
graph.get_schema
```

```output
在下面，每个IRI后面跟着本地名称，可选地跟着括号中的描述。
RDF图支持以下节点类型：
<http://xmlns.com/foaf/0.1/PersonalProfileDocument> (PersonalProfileDocument, None), <http://www.w3.org/ns/auth/cert#RSAPublicKey> (RSAPublicKey, None), <http://www.w3.org/2000/10/swap/pim/contact#Male> (Male, None), <http://xmlns.com/foaf/0.1/Person> (Person, None), <http://www.w3.org/2006/vcard/ns#Work> (Work, None)
RDF图支持以下关系：
<http://www.w3.org/2000/01/rdf-schema#seeAlso> (seeAlso, None), <http://purl.org/dc/elements/1.1/title> (title, None), <http://xmlns.com/foaf/0.1/mbox_sha1sum> (mbox_sha1sum, None), <http://xmlns.com/foaf/0.1/maker> (maker, None), <http://www.w3.org/ns/solid/terms#oidcIssuer> (oidcIssuer, None), <http://www.w3.org/2000/10/swap/pim/contact#publicHomePage> (publicHomePage, None), <http://xmlns.com/foaf/0.1/openid> (openid, None), <http://www.w3.org/ns/pim/space#storage> (storage, None), <http://xmlns.com/foaf/0.1/name> (name, None), <http://www.w3.org/2000/10/swap/pim/contact#country> (country, None), <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> (type, None), <http://www.w3.org/ns/solid/terms#profileHighlightColor> (profileHighlightColor, None), <http://www.w3.org/ns/pim/space#preferencesFile> (preferencesFile, None), <http://www.w3.org/2000/01/rdf-schema#label> (label, None), <http://www.w3.org/ns/auth/cert#modulus> (modulus, None), <http://www.w3.org/2000/10/swap/pim/contact#participant> (participant, None), <http://www.w3.org/2000/10/swap/pim/contact#street2> (street2, None), <http://www.w3.org/2006/vcard/ns#locality> (locality, None), <http://xmlns.com/foaf/0.1/nick> (nick, None), <http://xmlns.com/foaf/0.1/homepage> (homepage, None), <http://creativecommons.org/ns#license> (license, None), <http://xmlns.com/foaf/0.1/givenname> (givenname, None), <http://www.w3.org/2006/vcard/ns#street-address> (street-address, None), <http://www.w3.org/2006/vcard/ns#postal-code> (postal-code, None), <http://www.w3.org/2000/10/swap/pim/contact#street> (street, None), <http://www.w3.org/2003/01/geo/wgs84_pos#lat> (lat, None), <http://xmlns.com/foaf/0.1/primaryTopic> (primaryTopic, None), <http://www.w3.org/2006/vcard/ns#fn> (fn, None), <http://www.w3.org/2003/01/geo/wgs84_pos#location> (location, None), <http://usefulinc.com/ns/doap#developer> (developer, None), <http://www.w3.org/2000/10/swap/pim/contact#city> (city, None), <http://www.w3.org/2006/vcard/ns#region> (region, None), <http://xmlns.com/foaf/0.1/member> (member, None), <http://www.w3.org/2003/01/geo/wgs84_pos#long> (long, None), <http://www.w3.org/2000/10/swap/pim/contact#address> (address, None), <http://xmlns.com/foaf/0.1/family_name> (family_name, None), <http://xmlns.com/foaf/0.1/account> (account, None), <http://xmlns.com/foaf/0.1/workplaceHomepage> (workplaceHomepage, None), <http://purl.org/dc/terms/title> (title, None), <http://www.w3.org/ns/solid/terms#publicTypeIndex> (publicTypeIndex, None), <http://www.w3.org/2000/10/swap/pim/contact#office> (office, None), <http://www.w3.org/2000/10/swap/pim/contact#homePage> (homePage, None), <http://xmlns.com/foaf/0.1/mbox> (mbox, None), <http://www.w3.org/2000/10/swap/pim/contact#preferredURI> (preferredURI, None), <http://www.w3.org/ns/solid/terms#profileBackgroundColor> (profileBackgroundColor, None), <http://schema.org/owns> (owns, None), <http://xmlns.com/foaf/0.1/based_near> (based_near, None), <http://www.w3.org/2006/vcard/ns#hasAddress> (hasAddress, None), <http://xmlns.com/foaf/0.1/img> (img, None), <http://www.w3.org/2000/10/swap/pim/contact#assistant> (assistant, None), <http://xmlns.com/foaf/0.1/title> (title, None), <http://www.w3.org/ns/auth/cert#key> (key, None), <http://www.w3.org/ns/ldp#inbox> (inbox, None), <http://www.w3.org/ns/solid/terms#editableProfile> (editableProfile, None), <http://www.w3.org/2000/10/swap/pim/contact#postalCode> (postalCode, None), <http://xmlns.com/foaf/0.1/weblog> (weblog, None), <http://www.w3.org/ns/auth/cert#exponent> (exponent, None), <http://rdfs.org/sioc/ns#avatar> (avatar, None)
```

## 查询图谱

现在，您可以使用图谱 SPARQL QA 链来询问有关图谱的问题。

```python
chain = GraphSparqlQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True
)
```

```python
chain.run("Tim Berners-Lee的工作主页是什么？")
```

```output
> 进入新的GraphSparqlQAChain链...
识别到的意图:
SELECT
生成的SPARQL:
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?homepage
WHERE {
    ?person foaf:name "Tim Berners-Lee" .
    ?person foaf:workplaceHomepage ?homepage .
}
完整上下文:
[]
> 链结束。
```

```output
"Tim Berners-Lee的工作主页是 http://www.w3.org/People/Berners-Lee/。"
```

## 更新图谱

类似地，您可以使用自然语言更新图谱，即插入三元组。

```python
chain.run(
    "保存名为'Timothy Berners-Lee'的人在'http://www.w3.org/foo/bar/'有一个工作主页"
)
```

```output
> 进入新的GraphSparqlQAChain链...
识别到的意图:
UPDATE
生成的SPARQL:
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
INSERT {
    ?person foaf:workplaceHomepage <http://www.w3.org/foo/bar/> .
}
WHERE {
    ?person foaf:name "Timothy Berners-Lee" .
}
> 链结束。
```

```output
'成功地将三元组插入图谱。'
```

让我们验证结果：

```python
query = (
    """PREFIX foaf: <http://xmlns.com/foaf/0.1/>\n"""
    """SELECT ?hp\n"""
    """WHERE {\n"""
    """    ?person foaf:name "Timothy Berners-Lee" . \n"""
    """    ?person foaf:workplaceHomepage ?hp .\n"""
    """}"""
)
graph.query(query)
```

```output
[(rdflib.term.URIRef('https://www.w3.org/'),),
 (rdflib.term.URIRef('http://www.w3.org/foo/bar/'),)]
```

## 返回SQARQL查询

您可以使用 `return_sparql_query` 参数从 Sparql QA Chain 返回 SPARQL 查询步骤。

```python
chain = GraphSparqlQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, return_sparql_query=True
)
```

```python
result = chain("Tim Berners-Lee的工作主页是什么？")
print(f"SQARQL查询: {result['sparql_query']}")
print(f"最终答案: {result['result']}")
```

```output
> 进入新的GraphSparqlQAChain链...
识别到的意图:
SELECT
生成的SPARQL:
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?workHomepage
WHERE {
    ?person foaf:name "Tim Berners-Lee" .
    ?person foaf:workplaceHomepage ?workHomepage .
}
完整上下文:
[]
> 链结束。
SQARQL查询: PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?workHomepage
WHERE {
    ?person foaf:name "Tim Berners-Lee" .
    ?person foaf:workplaceHomepage ?workHomepage .
}
最终答案: Tim Berners-Lee的工作主页是 http://www.w3.org/People/Berners-Lee/。
```

```python
print(result["sparql_query"])
```

```output
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?workHomepage
WHERE {
    ?person foaf:name "Tim Berners-Lee" .
    ?person foaf:workplaceHomepage ?workHomepage .
}
```