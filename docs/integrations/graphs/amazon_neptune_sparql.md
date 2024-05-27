# Amazon Neptune与SPARQL

[Amazon Neptune](https://aws.amazon.com/neptune/)是一个高性能的图分析和无服务器数据库，具有出色的可伸缩性和可用性。

这个例子展示了使用`SPARQL`查询语言在`Amazon Neptune`图数据库中查询[资源描述框架（RDF）](https://en.wikipedia.org/wiki/Resource_Description_Framework)数据的问答链，然后返回人类可读的响应。

[SPARQL](https://en.wikipedia.org/wiki/SPARQL)是用于`RDF`图的标准查询语言。

这个例子使用了一个`NeptuneRdfGraph`类，它连接到Neptune数据库并加载其模式。`NeptuneSparqlQAChain`用于连接图形和LLM以提问自然语言问题。

这个笔记本演示了使用组织数据的示例。

运行此笔记本的要求：

- Neptune 1.2.x集群可从此笔记本访问

- 使用Python 3.9或更高版本的内核

- 对于Bedrock访问，请确保IAM角色具有此策略

```json
{
    "Action": [
        "bedrock:ListFoundationModels",
        "bedrock:InvokeModel"
    ],
    "Resource": "*",
    "Effect": "Allow"
}
```

- 用于暂存示例数据的S3存储桶。该存储桶应位于与Neptune相同的账户/区域中。

## 设置

### 种子W3C组织数据

种子W3C组织数据，W3C组织本体以及一些实例。

您将需要一个位于相同区域和账户中的S3存储桶。将`STAGE_BUCKET`设置为该存储桶的名称。

```python
STAGE_BUCKET = "<bucket-name>"
```

```bash
%%bash  -s "$STAGE_BUCKET"
rm -rf data
mkdir -p data
cd data
echo getting org ontology and sample org instances
wget http://www.w3.org/ns/org.ttl 
wget https://raw.githubusercontent.com/aws-samples/amazon-neptune-ontology-example-blog/main/data/example_org.ttl 
echo Copying org ttl to S3
aws s3 cp org.ttl s3://$1/org.ttl
aws s3 cp example_org.ttl s3://$1/example_org.ttl
```

批量加载组织ttl - 本体和实例

```python
%load -s s3://{STAGE_BUCKET} -f turtle --store-to loadres --run
```

```python
%load_status {loadres['payload']['loadId']} --errors --details
```

### 设置链

```python
!pip install --upgrade --quiet langchain langchain-community langchain-aws
```

**重新启动内核**

### 准备一个示例

```python
EXAMPLES = """
<question>
Find organizations.
</question>
<sparql>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX org: <http://www.w3.org/ns/org#> 
select ?org ?orgName where {
    ?org rdfs:label ?orgName .
} 
</sparql>
<question>
Find sites of an organization
</question>
<sparql>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX org: <http://www.w3.org/ns/org#> 
select ?org ?orgName ?siteName where {
    ?org rdfs:label ?orgName .
    ?org org:hasSite/rdfs:label ?siteName . 
} 
</sparql>
<question>
Find suborganizations of an organization
</question>
<sparql>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX org: <http://www.w3.org/ns/org#> 
select ?org ?orgName ?subName where {
    ?org rdfs:label ?orgName .
    ?org org:hasSubOrganization/rdfs:label ?subName  .
} 
</sparql>
<question>
Find organizational units of an organization
</question>
<sparql>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX org: <http://www.w3.org/ns/org#> 
select ?org ?orgName ?unitName where {
    ?org rdfs:label ?orgName .
    ?org org:hasUnit/rdfs:label ?unitName . 
} 
</sparql>
<question>
Find members of an organization. Also find their manager, or the member they report to.
</question>
<sparql>
PREFIX org: <http://www.w3.org/ns/org#> 
PREFIX foaf: <http://xmlns.com/foaf/0.1/> 
select * where {
    ?person rdf:type foaf:Person .
    ?person  org:memberOf ?org .
    OPTIONAL { ?person foaf:firstName ?firstName . }
    OPTIONAL { ?person foaf:family_name ?lastName . }
    OPTIONAL { ?person  org:reportsTo ??manager } .
}
</sparql>
<question>
Find change events, such as mergers and acquisitions, of an organization
</question>
<sparql>
PREFIX org: <http://www.w3.org/ns/org#> 
select ?event ?prop ?obj where {
    ?org rdfs:label ?orgName .
    ?event rdf:type org:ChangeEvent .
    ?event org:originalOrganization ?origOrg .
    ?event org:resultingOrganization ?resultingOrg .
}
</sparql>
"""
```

```python
import boto3
from langchain.chains.graph_qa.neptune_sparql import NeptuneSparqlQAChain
from langchain_aws import ChatBedrock
from langchain_community.graphs import NeptuneRdfGraph
host = "<your host>"
port = 8182  # change if different
region = "us-east-1"  # change if different
graph = NeptuneRdfGraph(host=host, port=port, use_iam_auth=True, region_name=region)
# Optionally change the schema
# elems = graph.get_schema_elements
# change elems ...
# graph.load_schema(elems)
MODEL_ID = "anthropic.claude-v2"
bedrock_client = boto3.client("bedrock-runtime")
llm = ChatBedrock(model_id=MODEL_ID, client=bedrock_client)
chain = NeptuneSparqlQAChain.from_llm(
    llm=llm,
    graph=graph,
    examples=EXAMPLES,
    verbose=True,
    top_K=10,
    return_intermediate_steps=True,
    return_direct=False,
)
```

## 提问

取决于我们上面摄取的数据

```python
chain.invoke("""图谱中有多少个组织""")
```

```python
chain.invoke("""是否有任何合并或收购""")
```

```python
chain.invoke("""查找组织""")
```

```python
chain.invoke("""查找 MegaSystems 或 MegaFinancial 的站点""")
```

```python
chain.invoke("""查找一个或多个成员的经理""")
```

```python
chain.invoke("""查找五个成员及其经理是谁""")
```

```python
chain.invoke(
    """查找 The Mega Group 的组织单位或子组织。这些单位的站点是什么？"""
)
```