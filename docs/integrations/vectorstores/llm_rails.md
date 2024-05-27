# LLMRails

>[LLMRails](https://www.llmrails.com/) 是一个用于构建 GenAI 应用程序的 API 平台。它提供了一个易于使用的 API，用于文档索引和查询，由 LLMRails 管理，并针对性能和准确性进行了优化。
有关如何使用 API 的更多信息，请参阅 [LLMRails API 文档](https://docs.llmrails.com/)。

本笔记本展示了与 `LLMRails` 集成的 langchain 相关功能的使用方法。
请注意，与此类别中的许多其他集成不同，LLMRails 提供了一个端到端的托管服务，用于检索增强生成，其中包括：
1. 从文档文件中提取文本并将其分块为句子的方法。
2. 其自己的嵌入模型和向量存储 - 每个文本段都被编码为一个向量嵌入，并存储在 LLMRails 的内部向量存储中。
3. 一个查询服务，自动将查询编码为嵌入，并检索最相关的文本段（包括对 [混合搜索](https://docs.llmrails.com/datastores/search) 的支持）

所有这些功能都在此 LangChain 集成中得到支持。

# 设置

您需要一个 LLMRails 账户才能使用 LLMRails 与 LangChain。要开始使用，请按照以下步骤进行操作：
1. 如果您还没有 LLMRails 账户，请[注册](https://console.llmrails.com/signup)一个 LLMRails 账户。
2. 接下来，您需要创建用于访问 API 的 API 密钥。在语料库视图中点击 **"API Keys"** 标签，然后点击 **"Create API Key"** 按钮。给您的密钥取一个名称。点击 "Create key"，您现在有一个活动的 API 密钥。请保密此密钥。

要使用 LLMRails 和 LangChain，您需要拥有以下值：api_key。
您可以通过以下两种方式将它们提供给 LangChain：

1. 在环境中包含这两个变量：`LLM_RAILS_API_KEY`，`LLM_RAILS_DATASTORE_ID`。

> 例如，您可以使用 os.environ 和 getpass 设置这些变量，如下所示：

```python
import os
import getpass

os.environ["LLM_RAILS_API_KEY"] = getpass.getpass("LLMRails API Key:")
os.environ["LLM_RAILS_DATASTORE_ID"] = getpass.getpass("LLMRails Datastore Id:")
```

2. 在创建 LLMRails vectorstore 对象时将它们作为参数提供：

```python
vectorstore = LLMRails(
    api_key=llm_rails_api_key,
    datastore_id=datastore_id
)
```

## 添加文本

要将文本添加到您的数据存储中，首先需要转到 [Datastores](https://console.llmrails.com/datastores) 页面并创建一个数据存储。点击 "Create Datastore" 按钮，选择一个名称和嵌入模型为您的数据存储。然后从新创建的数据存储设置中获取您的数据存储 id。

```python
%pip install tika
```
```output
Collecting tika
  Downloading tika-2.6.0.tar.gz (27 kB)
  Preparing metadata (setup.py) ... [?25ldone
[?25hRequirement already satisfied: setuptools in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from tika) (68.2.2)
Requirement already satisfied: requests in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from tika) (2.31.0)
Requirement already satisfied: charset-normalizer<4,>=2 in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from requests->tika) (2.1.1)
Requirement already satisfied: idna<4,>=2.5 in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from requests->tika) (3.4)
Requirement already satisfied: urllib3<3,>=1.21.1 in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from requests->tika) (1.26.16)
Requirement already satisfied: certifi>=2017.4.17 in /Users/omaraly/anaconda3/lib/python3.11/site-packages (from requests->tika) (2022.12.7)
Building wheels for collected packages: tika
  Building wheel for tika (setup.py) ... [?25ldone
[?25h  Created wheel for tika: filename=tika-2.6.0-py3-none-any.whl size=32621 sha256=b3f03c9dbd7f347d712c49027704d48f1a368f31560be9b4ee131f79a52e176f
  Stored in directory: /Users/omaraly/Library/Caches/pip/wheels/27/ba/2f/37420d1191bdae5e855d69b8e913673045bfd395cbd78ad697
Successfully built tika
Installing collected packages: tika
Successfully installed tika-2.6.0

[notice] A new release of pip is available: 23.3.1 -> 23.3.2
[notice] To update, run: pip install --upgrade pip
Note: you may need to restart the kernel to use updated packages.
```

```python
import os

from langchain_community.vectorstores import LLMRails

os.environ["LLM_RAILS_DATASTORE_ID"] = "您的数据存储 id"
os.environ["LLM_RAILS_API_KEY"] = "您的 API 密钥"

llm_rails = LLMRails.from_texts(["您的文本在这里"])
```

## 相似性搜索

使用 LLMRails 的最简单场景是执行相似性搜索。

```python
query = "您打算如何处理国家安全问题？"
found_docs = llm_rails.similarity_search(query, k=5)
```


```python
print(found_docs[0].page_content)
```
```output
6  国 家 安 全 战 略 第 7 页 

这份国家安全战略阐明了我们实现一个自由、开放、安全和繁荣世界更美好未来的计划。

我们的战略根植于我们的国家利益：保护美国人民的安全；扩大经济繁荣和机遇；实现并捍卫美国生活方式核心的民主价值观。

我们无法独自完成这一切，也无需独自完成。

世界上大多数国家以与我们相容的方式定义自己的利益。

我们将建立最强大、最广泛的可能的国家联盟，这些国家寻求相互合作，同时与那些提供更黑暗愿景的大国竞争，并阻止它们威胁我们的利益。

我们的永恒角色 在世界上扮演强大而有目的的美国角色的需求从未如此迫切。

世界变得更加分裂和不稳定。

自新冠疫情爆发以来，全球通货膨胀的增加使许多人的生活变得更加困难。

规范国家间关系的基本法律和原则，包括《联合国宪章》及其保护所有国家免受邻国入侵或以武力重新划定边界的规定，正遭受攻击。

主要大国之间发生冲突的风险正在增加。

民主国家和专制国家正参与一场竞赛，展示哪种治理体系能够最好地为他们的人民和世界提供服务。

竞争开发和部署将改变我们安全和经济的基础性技术正日益激烈。

全球在共同利益上的合作已经疲弱，尽管这种合作的必要性变得至关重要。

这些变化的规模随着每一年的过去而增加，不作为的风险也随之增加。

尽管国际环境变得更具竞争性，但美国仍然是世界领先大国。
```
## 带有分数的相似性搜索

有时候我们可能希望进行搜索，同时也获得一个相关性分数，以了解特定结果的好坏程度。

```python
query = "What is your approach to national defense"
found_docs = llm_rails.similarity_search_with_score(
    query,
    k=5,
)
```

```python
document, score = found_docs[0]
print(document.page_content)
print(f"\nScore: {score}")
```

```output
但我们将把这作为最后的手段，只有在目标和任务明确可行，与我们的价值观和法律一致，以及非军事工具一起使用，并且任务得到了美国人民的知情同意时，我们才会这样做。

我们对国家防御的方法在2022年国家防御战略中有详细描述。

我们的出发点是，强大的美国军队通过支持外交、对抗侵略、威慑冲突、展示实力以及保护美国人民及其经济利益，有助于推进和维护重要的美国国家利益。

在竞争日益激烈的环境中，军队的角色是在限制我们竞争对手的战斗优势的同时，维持和获得自身的战斗优势。

军队将紧急行动起来，以维持和加强威慑，以中华人民共和国作为其步调挑战。

我们将在国家防御方面做出有纪律的选择，并将注意力集中在军队的主要职责上：保卫国土，威慑对美国、我们的盟友和伙伴的攻击和侵略，同时准备在外交和威慑失败时打赢国家战争。

为此，我们将整合我们的优势，以在威慑侵略行为方面取得最大效果——我们将这种方法称为综合威慑（见第22页的文本框）。

我们将以战役思维方式运用我们的军队——逻辑上链接的军事活动序列，以推进与战略一致的优先事项。

此外，我们将建立一个有弹性的力量和防御生态系统，以确保我们能够在未来几十年内履行这些职能。

我们结束了美国在阿富汗的最长战争，也结束了重塑其他社会的主要军事行动的时代，同时我们仍然保持着应对恐怖威胁的能力，以保护美国人民。

20  国家安全战略 第21页

一个具备战斗能力的军队是威慑力量的基础，也是美国在冲突中取得胜利的能力。

分数：0.5040982687179959
```

## LLMRails 作为检索器

LLMRails，像其他所有的LangChain向量存储一样，最常用作LangChain检索器：

```python
retriever = llm_rails.as_retriever()
retriever
```

```output
LLMRailsRetriever(vectorstore=<langchain_community.vectorstores.llm_rails.LLMRails object at 0x1235b0e50>)
```

```python
query = "What is your approach to national defense"
retriever.invoke(query)
```
```output
[文档(page_content='但我们将把这作为最后的手段，只有在目标和任务明确可行、符合我们的价值观和法律，并且在非军事工具的支持下，以及在得到美国人民知情同意的情况下，才会这样做。\n\n我们对国家防御的方法在2022年国家防御战略中有详细描述。\n\n我们的出发点是，强大的美国军队通过支持外交、对抗侵略、威慑冲突、展示实力以及保护美国人民及其经济利益，有助于推进和维护重要的美国国家利益。\n\n在竞争日益激烈的环境中，军队的角色是在限制我们竞争对手的战斗优势的同时，保持和获得战斗优势。\n\n军队将紧急行动起来，以维持和加强威慑，以中华人民共和国作为其主要挑战。\n\n我们将在国家防御方面做出明智的选择，并将注意力集中在军队的主要责任上：保卫国土，威慑对美国、我们的盟友和伙伴的攻击和侵略，同时准备好在外交和威慑失败时打赢国家的战争。\n\n为此，我们将整合我们的力量，以实现最大的威慑效果——我们称之为综合威慑的方法（见第22页的文本框）。\n\n我们将以战役思维来运用我们的军队——逻辑上相互关联的军事活动来推进与战略一致的优先事项。\n\n我们将建立一个有弹性的力量和防御生态系统，以确保我们能够在未来几十年内履行这些职责。\n\n我们结束了美国在阿富汗的最长战争，也结束了通过军事行动改造其他社会的时代，同时我们仍然保持应对恐怖威胁的能力，以保护美国人民。\n\n20  国家安全战略 第21页 \x90\x90\x90\x90\x90\x90\n\n一个具有战斗能力的军队是威慑的基础，也是美国在冲突中取得胜利的能力。', metadata={'type': 'file', 'url': 'https://cdn.llmrails.com/dst_466092be-e79a-49f3-b3e6-50e51ddae186/a63892afdee3469d863520351bd5af9f', 'name': 'Biden-Harris-Administrations-National-Security-Strategy-10.2022.pdf', 'filters': {}}),
 文档(page_content='在这里输入您的文本', metadata={'type': 'text', 'url': 'https://cdn.llmrails.com/dst_466092be-e79a-49f3-b3e6-50e51ddae186/63c17ac6395e4be1967c63a16356818e', 'name': '71370a91-7f58-4cc7-b2e7-546325960330', 'filters': {}}),
 文档(page_content='第1页 国家安全战略 2022年10月 第2页 2022年10月12日，从我担任总统的最早时刻起，我就认为我们的世界正处于一个转折点。\n\n我们如何应对我们今天面临的巨大挑战和前所未有的机遇，将决定我们世界的走向，影响美国人民的安全和繁荣，影响未来几代人。\n\n2022年国家安全战略概述了我政府如何抓住这个决定性的十年，推进美国的重要利益，使美国能够在地缘政治竞争中超越我们的地缘政治竞争对手，应对共同挑战，并将我们的世界牢牢地引向更加光明和更有希望的未来。\n\n在世界各地，对美国领导力的需求与以往一样迫切。\n\n我们正处于一个塑造国际秩序未来的战略竞争之中。\n\n与此同时，影响全球各地人民的共同挑战要求加强全球合作，各国在这个时刻承担起责任变得更加困难。\n\n作为回应，美国将以我们的价值观为先导，我们将与我们的盟友和伙伴以及与我们共享利益的所有人紧密合作。\n\n我们不会让我们的未来受到那些不与我们共享自由、开放、繁荣和安全世界愿景的人的任性行为的威胁。\n\n随着世界继续应对大流行病和全球经济不确定性的影响，没有哪个国家比美利坚合众国更有能力以坚定和有目标的方式领导。\n\n从我宣誓就职的那一刻起，我的政府就专注于投资于美国的核心战略优势。\n\n我们的经济新增了1000万个就业岗位，失业率接近历史最低水平。\n\n制造业就业机会已经回流到美国。\n\n我们正在从底层和中间层重建我们的经济。', metadata={'type': 'file', 'url': 'https://cdn.llmrails.com/dst_466092be-e79a-49f3-b3e6-50e51ddae186/a63892afdee3469d863520351bd5af9f', 'name': 'Biden-Harris-Administrations-National-Security-Strategy-10.2022.pdf', 'filters': {}}),
 文档(page_content='在这里输入您的文本', metadata={'type': 'text', 'url': 'https://cdn.llmrails.com/dst_466092be-e79a-49f3-b3e6-50e51ddae186/8c414a9306e04d47a300f0289ba6e9cf', 'name': 'dacc29f5-8c63-46e0-b5aa-cab2d3c99fb7', 'filters': {}}),
 文档(page_content='为了确保我们的核威慑能力能够应对我们面临的威胁，我们正在现代化核三位一体、核指挥、控制和通信以及我们的核武器基础设施，并加强我们对盟友的扩展威慑承诺。\n\n我们同样致力于减少核战争的风险。\n\n这包括进一步减少核武器在我们战略中的作用，并追求相互可验证的现实目标，这有助于我们的威慑战略，并加强全球防扩散体系。\n\n最重要的投资是在陆军、海军陆战队、海军、空军、太空部队、海岸警卫队以及我们国防部的文职
## 什么是无损压缩？

无损压缩是一种压缩数据的方法，它可以减小文件的大小，但不会损失任何数据。这意味着压缩后的文件可以完全恢复为原始文件，而不会丢失任何信息。

在无损压缩中，压缩算法会利用数据中的冗余和重复性来减小文件的大小。这些算法会找到数据中的模式，并使用更简洁的方式来表示这些模式。例如，如果一个文件中有很多连续的相同的字节，无损压缩算法可以使用一个字节来表示这个模式，而不是重复写入多个相同的字节。

无损压缩广泛应用于各种领域，包括图像、音频和视频压缩。常见的无损压缩算法包括FLAC（Free Lossless Audio Codec）用于音频压缩，以及PNG（Portable Network Graphics）和JPEG（Joint Photographic Experts Group）用于图像压缩。

无损压缩的一个重要应用是在网络传输中减小文件的大小。通过使用无损压缩算法，可以更快地传输文件，并减少网络带宽的使用。此外，无损压缩还可以节省存储空间，使得文件更容易管理和备份。

虽然无损压缩可以减小文件的大小，但它通常不能达到与有损压缩相同的压缩比。有损压缩是另一种压缩数据的方法，它可以更大幅度地减小文件的大小，但会损失一些数据质量。因此，在选择压缩方法时，需要权衡文件大小和数据质量之间的平衡。