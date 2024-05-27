# Supabase (Postgres)

[Supabase](https://supabase.com/docs) 是一个开源的 Firebase 替代品。`Supabase` 建立在 `PostgreSQL` 之上，后者提供了强大的 SQL 查询功能，并能够与已有的工具和框架简单地进行接口对接。

[PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) 也被称为 `Postgres`，是一个强调可扩展性和 SQL 兼容性的免费开源关系数据库管理系统（RDBMS）。

这个笔记本展示了如何将 `Supabase` 和 `pgvector` 用作您的 VectorStore。

要运行此笔记本，请确保：

- `pgvector` 扩展已启用

- 您已安装了 `supabase-py` 包

- 您在数据库中已创建了一个 `match_documents` 函数

- 您在 `public` 模式下有一个类似下面的 `documents` 表。

以下函数确定余弦相似度，但您可以根据需要进行调整。

```sql
-- 启用 pgvector 扩展以处理嵌入向量
create extension if not exists vector;
-- 创建一个表来存储您的文档
create table
  documents (
    id uuid primary key,
    content text, -- 对应 Document.pageContent
    metadata jsonb, -- 对应 Document.metadata
    embedding vector (1536) -- 1536 适用于 OpenAI 嵌入，如有需要可更改
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

```python
# 使用 pip
%pip install --upgrade --quiet  supabase
# 使用 conda
# !conda install -c conda-forge supabase
```

我们想要使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
os.environ["SUPABASE_URL"] = getpass.getpass("Supabase URL:")
```

```python
os.environ["SUPABASE_SERVICE_KEY"] = getpass.getpass("Supabase Service Key:")
```

```python
# 如果您将 Supabase 和 OpenAI API 密钥存储在 .env 文件中，可以使用 dotenv 加载它们
from dotenv import load_dotenv
load_dotenv()
```

首先，我们将创建一个 Supabase 客户端并实例化一个 OpenAI 嵌入类。

```python
import os
from langchain_community.vectorstores import SupabaseVectorStore
from langchain_openai import OpenAIEmbeddings
from supabase.client import Client, create_client
supabase_url = os.environ.get("SUPABASE_URL")
supabase_key = os.environ.get("SUPABASE_SERVICE_KEY")
supabase: Client = create_client(supabase_url, supabase_key)
embeddings = OpenAIEmbeddings()
```

接下来，我们将加载并解析一些数据到我们的向量存储中（如果您的数据库中已经存储了带有嵌入的文档，则可以跳过此步骤）。

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

将上述文档插入数据库。每个文档将自动生成嵌入。您可以根据文档数量调整 `chunk_size`。默认值为 500，但可能需要降低它。

```python
vector_store = SupabaseVectorStore.from_documents(
    docs,
    embeddings,
    client=supabase,
    table_name="documents",
    query_name="match_documents",
    chunk_size=500,
)
```

或者，如果您的数据库中已经有带有嵌入的文档，只需直接实例化一个新的 `SupabaseVectorStore`：

```python
vector_store = SupabaseVectorStore(
    embedding=embeddings,
    client=supabase,
    table_name="documents",
    query_name="match_documents",
)
```

最后，通过执行相似度搜索来测试它：

```python
query = "What did the president say about Ketanji Brown Jackson"
matched_docs = vector_store.similarity_search(query)
```

```python
print(matched_docs[0].page_content)
```

```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## 使用分数进行相似度搜索

返回的距离分数是余弦距离。因此，得分越低越好。

```python
matched_docs = vector_store.similarity_search_with_relevance_scores(query)
```

```python
matched_docs[0]
```

```output
(Document(page_content='今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯选举权法案》。还有，通过《揭露法案》，这样美国人就可以知道是谁在资助我们的选举。\n\n今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。\n\n总统最严肃的宪法责任之一就是提名人选担任美国最高法院法官。\n\n而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶司法部长的卓越传统。', metadata={'source': '../../../state_of_the_union.txt'}), 0.802509746274066)
```

## 检索器选项

本节介绍了如何使用 SupabaseVectorStore 作为检索器的不同选项。

### 最大边际相关搜索

除了在检索器对象中使用相似度搜索外，您还可以使用 `mmr`。

```python
retriever = vector_store.as_retriever(search_type="mmr")
```

```python
matched_docs = retriever.invoke(query)
```

```python
for i, d in enumerate(matched_docs):
    print(f"\n## 文档 {i}\n")
    print(d.page_content)
```

```output
## 文档 0
今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯选举权法案》。还有，通过《揭露法案》，这样美国人就可以知道是谁在资助我们的选举。
今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一就是提名人选担任美国最高法院法官。
而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶司法部长的卓越传统。
## 文档 1
有人驻扎在基地，呼吸着“燃烧坑”散发的有毒烟雾——焚烧战争废物、医疗和危险材料、喷气燃料等。
他们回家后，许多世界上最健壮、训练最好的战士再也不同往日。
头痛。麻木。头晕。
一种癌症让他们躺在用国旗覆盖的棺材里。
我知道。
其中一名士兵是我的儿子博大校尉。
我们不确定燃烧坑是否导致了他的脑癌，或者导致了我们的许多军队人员的疾病。
但我承诺要尽一切努力找出我们能知道的一切。
致力于像俄亥俄州的丹尼尔·罗宾逊这样的军事家庭。
他是希斯·罗宾逊一等兵的遗孀。
他生来就是一名士兵。国民警卫队。科索沃和伊拉克的战地医生。
驻扎在距离足球场大小的燃烧坑几码远的巴格达附近。
希斯的遗孀丹尼尔今晚和我们在一起。他们喜欢去俄亥俄州立大学的橄榄球比赛。他喜欢和女儿一起搭积木。
## 文档 2
我正在采取有力措施，确保我们的制裁痛击俄罗斯的经济。我将利用我们掌握的一切工具来保护美国企业和消费者。
今晚，我可以宣布，美国已与其他30个国家合作，从世界各地的储备释放了6000万桶石油。
美国将领导这一努力，从我们自己的战略石油储备中释放3000万桶。如果有必要，我们愿意做更多，与我们的盟友团结一致。
这些举措将有助于减轻国内的汽油价格。我知道关于正在发生的事情的消息似乎令人担忧。
但我想让你知道，我们会没事的。
当这个时代的历史被书写时，普京对乌克兰的战争将使俄罗斯变得更加脆弱，而世界其他地方将变得更加强大。
尽管人们不应该等到发生如此可怕的事情才能看清现在的利害关系，但现在每个人都清楚地看到了。
```

## 结束

以上就是对英文段落的中文翻译，希望对您有所帮助。

两位多米尼加裔美国人，在同一条街道长大，后来选择成为警察。

我与他们的家人交谈，并告诉他们我们永远感激他们的牺牲，我们将继续履行他们的使命，恢复每个社区应有的信任和安全。

我一直在处理这些问题很长时间了。

我知道什么是有效的：投资于犯罪预防和社区警务人员，他们会巡逻，了解社区，恢复信任和安全。