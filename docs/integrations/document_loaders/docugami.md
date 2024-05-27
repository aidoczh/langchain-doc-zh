# Docugami

本笔记介绍如何从 `Docugami` 加载文档。它提供了使用该系统相对于其他数据加载器的优势。

## 先决条件

1. 安装必要的 Python 包。

2. 获取工作区的访问令牌，并确保将其设置为 `DOCUGAMI_API_KEY` 环境变量。

3. 获取一些已处理文档的文档集和文档 ID，如此处所述：https://help.docugami.com/home/docugami-api

```python
# 您需要 dgml-utils 包来使用 DocugamiLoader（如果您没有使用 poetry，则直接运行 pip install 而不是 "poetry run"）
!poetry run pip install docugami-langchain dgml-utils==0.3.0 --upgrade --quiet
```

## 快速开始

1. 创建一个 [Docugami 工作区](http://www.docugami.com)（提供免费试用）

2. 添加您的文档（PDF、DOCX 或 DOC），并允许 Docugami 将其摄取并聚类成相似文档集，例如保密协议、租赁协议和服务协议。系统不支持固定的文档类型集，创建的聚类取决于您的特定文档，您可以稍后 [更改文档集分配](https://help.docugami.com/home/working-with-the-doc-sets-view)。

3. 通过开发者游乐场为您的工作区创建一个访问令牌。[详细说明](https://help.docugami.com/home/docugami-api)

4. 探索 [Docugami API](https://api-docs.docugami.com)，以获取已处理的文档集 ID 列表，或者特定文档集的文档 ID 列表。

6. 使用下面详细说明的 DocugamiLoader，获取文档的丰富语义块。

7. 可选地，构建并发布一个或多个 [报告或摘要](https://help.docugami.com/home/reports)。这有助于 Docugami 根据您的偏好改进语义 XML，并将其作为元数据添加到 DocugamiLoader 输出中。使用诸如 [自查询检索器](/docs/how_to/self_query) 之类的技术进行高准确度的文档质量保证。

## 与其他块技术相比的优势

适当的文档块对于从文档中检索信息至关重要。存在许多块技术，包括依赖空格和基于字符长度的递归块拆分的简单技术。Docugami 提供了一种不同的方法：

1. **智能块拆分：** Docugami 将每个文档分解为具有不同大小的层次化语义 XML 块树，从单词或数值到整个部分。这些块遵循文档的语义轮廓，提供比任意长度或简单基于空格的块拆分更有意义的表示。

2. **语义标注：** 块带有跨文档集一致的语义标记，促进跨多个文档的一致层次化查询，即使这些文档是以不同的方式编写和格式化的。例如，在租赁协议集中，您可以轻松识别关键条款，如房东、租户或续约日期，以及更复杂的信息，如任何转租条款的措辞或特定司法管辖区是否在终止条款中有例外部分。

3. **结构化表示：** 此外，XML 树使用属性表示每个文档的结构轮廓，包括标题、段落、列表、表格和其他常见元素，并且在所有支持的文档格式（如扫描的 PDF 或 DOCX 文件）中保持一致。它适当处理长篇文档特征，如页面页眉/页脚或多列流以进行清晰的文本提取。

4. **附加元数据：** 如果用户一直在使用 Docugami，块还将带有附加元数据。可以使用此附加元数据进行无上下文窗口限制的高准确度文档质量保证。请参阅下面的详细代码演示。

```python
import os
from docugami_langchain.document_loaders import DocugamiLoader
```

## 加载文档

如果已设置 `DOCUGAMI_API_KEY` 环境变量，则无需显式将其传递给加载程序，否则可以将其作为 `access_token` 参数传递。

```python
DOCUGAMI_API_KEY = os.environ.get("DOCUGAMI_API_KEY")
```
```python
docset_id = "26xpy3aes7xp"
document_ids = ["d7jqdzcj50sj", "cgd1eacfkchw"]
# 要加载给定文档集 ID 中的所有文档，只需不提供 document_ids
loader = DocugamiLoader(docset_id=docset_id, document_ids=document_ids)
chunks = loader.load()
len(chunks)
```
```output
120
```

每个 `Document`（实际上是 PDF、DOC 或 DOCX 的一个块）的 `metadata` 包含一些有用的附加信息：

1. **id 和 source：** 块来源文件（PDF、DOC 或 DOCX）的 ID 和名称，位于 Docugami 中。

2. **xpath：** 块在文档的 XML 表示中的 XPath。对于直接引用文档 XML 中的实际块很有用。

3. **结构：**块的结构属性，例如h1、h2、div、table、td等。如果需要，调用者可以使用这些属性来过滤出特定类型的块。

4. **标签：**块的语义标签，使用各种生成和抽取技术。更多详细信息请参见：[https://github.com/docugami/DFM-benchmarks](https://github.com/docugami/DFM-benchmarks)

您可以通过在`DocugamiLoader`实例上设置以下属性来控制块化行为：

1. 您可以设置最小和最大块大小，系统会尽量遵守这些大小，并尽量减少截断。您可以设置`loader.min_text_length`和`loader.max_text_length`来控制这些大小。

2. 默认情况下，只返回块的文本。然而，Docugami的XML知识图中还包含了块内实体的语义标签等其他丰富信息。如果您希望在返回的块上获取额外的XML元数据，请设置`loader.include_xml_tags = True`。

3. 此外，如果您希望Docugami返回父块和其返回的块一起返回，您可以设置`loader.parent_hierarchy_levels`。子块通过`loader.parent_id_key`值指向父块。这在使用[MultiVector Retriever](/docs/how_to/multi_vector)进行[small-to-big](https://www.youtube.com/watch?v=ihSiRrOUwmg)检索时非常有用。稍后在本笔记本中会有详细的示例。

```python
loader.min_text_length = 64
loader.include_xml_tags = True
chunks = loader.load()
for chunk in chunks[:5]:
    print(chunk)
```
```output
page_content='MASTER SERVICES AGREEMENT\n <ThisServicesAgreement> This Services Agreement (the “Agreement”) sets forth terms under which <Company>MagicSoft, Inc. </Company>a <Org><USState>Washington </USState>Corporation </Org>(“Company”) located at <CompanyAddress><CompanyStreetAddress><Company>600 </Company><Company>4th Ave</Company></CompanyStreetAddress>, <Company>Seattle</Company>, <Client>WA </Client><ProvideServices>98104 </ProvideServices></CompanyAddress>shall provide services to <Client>Daltech, Inc.</Client>, a <Company><USState>Washington </USState>Corporation </Company>(the “Client”) located at <ClientAddress><ClientStreetAddress><Client>701 </Client><Client>1st St</Client></ClientStreetAddress>, <Client>Kirkland</Client>, <State>WA </State><Client>98033</Client></ClientAddress>. This Agreement is effective as of <EffectiveDate>February 15, 2021 </EffectiveDate>(“Effective Date”). </ThisServicesAgreement>' metadata={'xpath': '/dg:chunk/docset:MASTERSERVICESAGREEMENT-section/dg:chunk', 'id': 'c28554d0af5114e2b102e6fc4dcbbde5', 'name': 'Master Services Agreement - Daltech.docx', 'source': 'Master Services Agreement - Daltech.docx', 'structure': 'h1 p', 'tag': 'chunk ThisServicesAgreement', 'Liability': '', 'Workers Compensation Insurance': '$1,000,000', 'Limit': '$1,000,000', 'Commercial General Liability Insurance': '$2,000,000', 'Technology Professional Liability Errors Omissions Policy': '$5,000,000', 'Excess Liability Umbrella Coverage': '$9,000,000', 'Client': 'Daltech, Inc.', 'Services Agreement Date': 'INITIAL STATEMENT  OF WORK (SOW)  The purpose of this SOW is to describe the Software and Services that Company will initially provide to  Daltech, Inc.  the “Client”) under the terms and conditions of the  Services Agreement  entered into between the parties on  June 15, 2021', 'Completion of the Services by Company Date': 'February 15, 2022', 'Charge': 'one hundred percent (100%)', 'Company': 'MagicSoft, Inc.', 'Effective Date': 'February 15, 2021', 'Start Date': '03/15/2021', 'Scheduled Onsite Visits Are Cancelled': 'ten (10) working days', 'Limit on Liability': '', 'Liability Cap': '', 'Business Automobile Liability': 'Business Automobile Liability  covering all vehicles that Company owns, hires or leases with a limit of no less than  $1,000,000  (combined single limit for bodily injury and property damage) for each accident.', 'Contractual Liability Coverage': 'Commercial General Liability insurance including  Contractual Liability Coverage , with coverage for products liability, completed operations, property damage and bodily injury, including  death , with an aggregate limit of no less than  $2,000,000 . This policy shall name Client as an additional insured with respect to the provision of services provided under this Agreement. This policy shall include a waiver of subrogation against Client.', 'Technology Professional Liability Errors Omissions': 'Technology Professional Liability Errors & Omissions policy (which includes Cyber Risk coverage and Computer Security and Privacy Liability coverage) with a limit of no less than  $5,000,000  per occurrence and in the aggregate.'}
page_content='A. STANDARD SOFTWARE AND SERVICES AGREEMENT\n 1. Deliverables.\n Company shall provide Client with software, technical support, product management, development, and <_testRef>testing </_testRef>services (“Services”) to the Client as described on one or more Statements of Work signed by Company and Client that reference this Agreement (“SOW” or “Statement of Work”). Company shall perform Services in a prompt manner and have the final product or service (“Deliverable”) ready for Client no later than the due date specified in the applicable SOW (“Completion Date”). This due date is subject to change in accordance with the Change Order process defined in the applicable SOW. Client shall assist Company by promptly providing all information requests known or available and relevant to the Services in a timely manner.' metadata={'xpath': '/dg:chunk/docset:MASTERSERVICESAGREEMENT-section/docset:MASTERSERVICESAGREEMENT/dg:chunk[1]/docset:Standard/dg:chunk[1]/dg:chunk[1]', 'id': 'de60160d328df10fa2637637c803d2d4', 'name': 'Master Services Agreement - Daltech.docx', 'source': 'Master Services Agreement - Daltech.docx', 'structure': 'lim h1 lim h1 div', 'tag': 'chunk', 'Liability': '', 'Workers Compensation Insurance': '$1,000,000', 'Limit': '$1,000,000', 'Commercial General Liability Insurance': '$2,000,000', 'Technology Professional Liability Errors Omissions Policy': '$5,000,000', 'Excess Liability Umbrella Coverage': '$9,000,000', 'Client': 'Daltech, Inc.', 'Services Agreement Date': 'INITIAL STATEMENT  OF WORK (SOW)  The purpose of this SOW is to describe the Software and Services that Company will initially provide to  Daltech, Inc.  the “Client”) under the terms and conditions of the  Services Agreement  entered into between the parties on  June 15, 2021', 'Completion of the Services by Company Date': 'February 15, 2022', 'Charge': 'one hundred percent (100%)', 'Company': 'MagicSoft, Inc.', 'Effective Date': 'February 15, 2021', 'Start Date': '03/15/2021', 'Scheduled Onsite Visits Are Cancelled': 'ten (10) working days', 'Limit on Liability': '', 'Liability Cap': '', 'Business Automobile Liability': 'Business Automobile Liability  covering all vehicles that Company owns, hires or leases with a limit of no less than  $1,000,000  (combined single limit for bodily injury and property damage) for each accident.', 'Contractual Liability Coverage': 'Commercial General Liability insurance including  Contractual Liability Coverage , with coverage for products liability, completed operations, property damage and bodily injury, including  death , with an aggregate limit of no less than  $2,000,000 . This policy shall name Client as an additional insured with respect to the provision of services provided under this Agreement. This policy shall include a waiver of subrogation against Client.', 'Technology Professional Liability Errors Omissions': 'Technology Professional Liability Errors & Omissions policy (which includes Cyber Risk coverage and Computer Security and Privacy Liability coverage) with a limit of no less than  $5,000,000  per occurrence and in the aggregate.'}
page_content='2. Onsite Services.\n 2.1 Onsite visits will be charged on a <Frequency>daily </Frequency>basis (minimum <OnsiteVisits>8 hours</OnsiteVisits>).' metadata={'xpath': '/dg:chunk/docset:MASTERSERVICESAGREEMENT-section/docset:MASTERSERVICESAGREEMENT/dg:chunk[1]/docset:Standard/dg:chunk[3]/dg:chunk[1]', 'id': 'db18315b437ac2de6b555d2d8ef8f893', 'name': 'Master Services Agreement - Daltech.docx', 'source': 'Master Services Agreement - Daltech.docx', 'structure': 'lim h1 lim p', 'tag': 'chunk', 'Liability': '', 'Workers Compensation Insurance': '$1,000,000', 'Limit': '$1,000,000', 'Commercial General Liability Insurance': '$2,000,000', 'Technology Professional Liability Errors Omissions Policy': '$5,000,000', 'Excess Liability Umbrella Coverage': '$9,000,000', 'Client': 'Daltech, Inc.', 'Services Agreement Date': 'INITIAL STATEMENT  OF WORK (SOW)  The purpose of this SOW is to describe the Software and Services that Company will initially provide to  Daltech, Inc.  the “Client”) under the terms and conditions of the  Services Agreement  entered into between the parties on  June 15, 2021', 'Completion of the Services by Company Date': 'February 15, 2022', 'Charge': 'one hundred percent (100%)', 'Company': 'MagicSoft, Inc.', 'Effective Date': 'February 15, 2021', 'Start Date': '03/15/2021', 'Scheduled Onsite Visits Are Cancelled': 'ten (10) working days', 'Limit on Liability': '', 'Liability Cap': '', 'Business Automobile Liability': 'Business Automobile Liability  covering all vehicles that Company owns, hires or leases with a limit of no less than  $1,000,000  (combined single limit for bodily injury and property damage) for each accident.', 'Contractual Liability Coverage': 'Commercial General Liability insurance including  Contractual Liability Coverage , with coverage for products liability, completed operations, property damage and bodily injury, including  death , with an aggregate limit of no less than  $2,000,000 . This policy shall name Client as an additional insured with respect to the provision of services provided under this Agreement. This policy shall include a waiver of subrogation against Client.', 'Technology Professional Liability Errors Omissions': 'Technology Professional Liability Errors & Omissions policy (which includes Cyber Risk coverage and Computer Security and Privacy Liability coverage) with a limit of no less than  $5,000,000  per occurrence and in the aggregate.'}
page_content='2.2 <Expenses>Time and expenses will be charged based on actuals unless otherwise described in an Order Form or accompanying SOW. </Expenses>' metadata={'xpath': '/dg:chunk/docset:MASTERSERVICESAGREEMENT-section/docset:MASTERSERVICESAGREEMENT/dg:chunk[1]/docset:Standard/dg:chunk[3]/dg:chunk[2]/docset:ADailyBasis/dg:chunk[2]/dg:chunk', 'id': '506220fa472d5c48c8ee3db78c1122c1', 'name': 'Master Services Agreement - Daltech.docx', 'source': 'Master Services Agreement - Daltech.docx', 'structure': 'lim p', 'tag': 'chunk Expenses', 'Liability': '', 'Workers Compensation Insurance': '$1,000,000', 'Limit': '$1,000,000', 'Commercial General Liability Insurance': '$2,000,000', 'Technology Professional Liability Errors Omissions Policy': '$5,000,000', 'Excess Liability Umbrella Coverage': '$9,000,000', 'Client': 'Daltech, Inc.', 'Services Agreement Date': 'INITIAL STATEMENT  OF WORK (SOW)  The purpose of this SOW is to describe the Software and Services that Company will initially provide to  Daltech, Inc.  the “Client”) under the terms and conditions of the  Services Agreement  entered into between the parties on  June 15, 2021', 'Completion of the Services by Company Date': 'February 15, 2022', 'Charge': 'one hundred percent (100%)', 'Company': 'MagicSoft, Inc.', 'Effective Date': 'February 15, 2021', 'Start Date': '03/15/2021', 'Scheduled Onsite Visits Are Cancelled': 'ten (10) working days', 'Limit on Liability': '', 'Liability Cap': '', 'Business Automobile Liability': 'Business Automobile Liability  covering all vehicles that Company owns, hires or leases with a limit of no less than  $1,000,000  (combined single limit for bodily injury and property damage) for each accident.', 'Contractual Liability Coverage': 'Commercial General Liability insurance including  Contractual Liability Coverage , with coverage for products liability, completed operations, property damage and bodily injury, including  death , with an aggregate limit of no less than  $2,000,000 . This policy shall name Client as an additional insured with respect to the provision of services provided under this Agreement. This policy shall include a waiver of subrogation against Client.', 'Technology Professional Liability Errors Omissions': 'Technology Professional Liability Errors & Omissions policy (which includes Cyber Risk coverage and Computer Security and Privacy Liability coverage) with a limit of no less than  $5,000,000  per occurrence and in the aggregate.'}
page_content='2.3 <RegularWorkingHours>All work will be executed during regular working hours <RegularWorkingHours>Monday</RegularWorkingHours>-<Weekday>Friday </Weekday><RegularWorkingHours><RegularWorkingHours>0800</RegularWorkingHours>-<Number>1900</Number></RegularWorkingHours>. For work outside of these hours on weekdays, Company will charge <Charge>one hundred percent (100%) </Charge>of the regular hourly rate and <Charge>two hundred percent (200%) </Charge>for Saturdays, Sundays and public holidays applicable to Company. </RegularWorkingHours>' metadata={'xpath': '/dg:chunk/docset:MASTERSERVICESAGREEMENT-section/docset:MASTERSERVICESAGREEMENT/dg:chunk[1]/docset:Standard/dg:chunk[3]/dg:chunk[2]/docset:ADailyBasis/dg:chunk[3]/dg:chunk', 'id': 'dac7a3ded61b5c4f3e59771243ea46c1', 'name': 'Master Services Agreement - Daltech.docx', 'source': 'Master Services Agreement - Daltech.docx', 'structure': 'lim p', 'tag': 'chunk RegularWorkingHours', 'Liability': '', 'Workers Compensation Insurance': '$1,000,000', 'Limit': '$1,000,000', 'Commercial General Liability Insurance': '$2,000,000', 'Technology Professional Liability Errors Omissions Policy': '$5,000,000', 'Excess Liability Umbrella Coverage': '$9,000,000', 'Client': 'Daltech, Inc.', 'Services Agreement Date': 'INITIAL STATEMENT  OF WORK (SOW)  The purpose of this SOW is to describe the Software and Services that Company will initially provide to  Daltech, Inc.  the “Client”) under the terms and conditions of the  Services Agreement  entered into between the parties on  June 15, 2021', 'Completion of the Services by Company Date': 'February 15, 2022', 'Charge': 'one hundred percent (100%)', 'Company': 'MagicSoft, Inc.', 'Effective Date': 'February 15, 2021', 'Start Date': '03/15/2021', 'Scheduled Onsite Visits Are Cancelled': 'ten (10) working days', 'Limit on Liability': '', 'Liability Cap': '', 'Business Automobile Liability': 'Business Automobile Liability  covering all vehicles that Company owns, hires or leases with a limit of no less than  $1,000,000  (combined single limit for bodily injury and property damage) for each accident.', 'Contractual Liability Coverage': 'Commercial General Liability insurance including  Contractual Liability Coverage , with coverage for products liability, completed operations, property damage and bodily injury, including  death , with an aggregate limit of no less than  $2,000,000 . This policy shall name Client as an additional insured with respect to the provision of services provided under this Agreement. This policy shall include a waiver of subrogation against Client.', 'Technology Professional Liability Errors Omissions': 'Technology Professional Liability Errors & Omissions policy (which includes Cyber Risk coverage and Computer Security and Privacy Liability coverage) with a limit of no less than  $5,000,000  per occurrence and in the aggregate.'}
```

## 基本用法：Docugami Loader 用于文档问答

你可以像使用标准的文档问答加载器一样使用 Docugami Loader，尽管它的分块要好得多，可以跟随文档的自然轮廓。有许多关于如何做到这一点的很棒的教程，例如[这个](https://www.youtube.com/watch?v=3yPBVii7Ct0)。我们可以使用相同的代码，但是使用 `DocugamiLoader` 来获得更好的分块，而不是直接使用基本的分割技术加载文本或 PDF 文件。

```python
!poetry run pip install --upgrade langchain-openai tiktoken chromadb hnswlib
```
```python
# 对于这个示例，我们已经有了一组租赁文件的处理文档集
loader = DocugamiLoader(docset_id="zo954yqy53wp")
chunks = loader.load()
# 故意剥离语义元数据，以测试没有语义元数据时的工作情况
for chunk in chunks:
    stripped_metadata = chunk.metadata.copy()
    for key in chunk.metadata:
        if key not in ["name", "xpath", "id", "structure"]:
            # 删除语义元数据
            del stripped_metadata[key]
    chunk.metadata = stripped_metadata
print(len(chunks))
```
```output
4674
```

加载器返回的文档已经被分割，因此我们不需要使用文本分割器。可选地，我们可以使用每个文档的元数据，例如结构或标签属性，进行任何我们想要的后处理。

我们将直接使用 `DocugamiLoader` 的输出来设置检索问答链。

```python
from langchain.chains import RetrievalQA
from langchain_community.vectorstores.chroma import Chroma
from langchain_openai import OpenAI, OpenAIEmbeddings
embedding = OpenAIEmbeddings()
vectordb = Chroma.from_documents(documents=chunks, embedding=embedding)
retriever = vectordb.as_retriever()
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(), chain_type="stuff", retriever=retriever, return_source_documents=True
)
```
```python
# 用一个示例查询尝试检索器
qa_chain("What can tenants do with signage on their properties?")
```
```output
{'query': 'What can tenants do with signage on their properties?',
 'result': ' Tenants can place or attach signage (digital or otherwise) to their property after receiving written permission from the landlord, which permission shall not be unreasonably withheld. The signage must conform to all applicable laws, ordinances, etc. governing the same, and tenants must remove all such signs by the termination of the lease.',
 'source_documents': [Document(page_content='6.01 Signage. Tenant may place or attach to the Premises signs (digital or otherwise) or other such identification as needed after receiving written permission from the Landlord, which permission shall not be unreasonably withheld. Any damage caused to the Premises by the Tenant’s erecting or removing such signs shall be repaired promptly by the Tenant at the Tenant’s expense. Any signs or other form of identification allowed must conform to all applicable laws, ordinances, etc. governing the same. Tenant also agrees to have any window or glass identification completely removed and cleaned at its expense promptly upon vacating the Premises. ARTICLE VII UTILITIES', metadata={'id': '1c290eea05915ba0f24c4a1ffc05d6f3', 'name': 'Sample Commercial Leases/TruTone Lane 6.pdf', 'structure': 'lim h1', 'xpath': '/dg:chunk/dg:chunk/dg:chunk[2]/dg:chunk[1]/docset:TheApprovedUse/dg:chunk[12]/dg:chunk[1]'}),
  Document(page_content='6.01 Signage. Tenant may place or attach to the Premises signs (digital or otherwise) or other such identification as needed after receiving written permission from the Landlord, which permission shall not be unreasonably withheld. Any damage caused to the Premises by the Tenant’s erecting or removing such signs shall be repaired promptly by the Tenant at the Tenant’s expense. Any signs or other form of identification allowed must conform to all applicable laws, ordinances, etc. governing the same. Tenant also agrees to have any window or glass identification completely removed and cleaned at its expense promptly upon vacating the Premises. ARTICLE VII UTILITIES', metadata={'id': '1c290eea05915ba0f24c4a1ffc05d6f3', 'name': 'Sample Commercial Leases/TruTone Lane 2.pdf', 'structure': 'lim h1', 'xpath': '/dg:chunk/dg:chunk/dg:chunk[2]/dg:chunk[1]/docset:TheApprovedUse/dg:chunk[12]/dg:chunk[1]'}),
```

## 使用 Docugami 知识图谱进行高准确性文档问答

大型文档的一个问题是，对于您的问题，正确答案可能取决于文档中相距较远的块。即使使用重叠的典型分块技术，也很难为 LLM 提供足够的上下文来回答此类问题。随着即将推出的非常大上下文 LLM，可能可以将大量令牌（甚至整个文档）放入上下文中，但是对于非常长的文档或大量文档，这仍然会达到某些限制。

例如，如果我们提出一个更复杂的问题，需要 LLM 从文档的不同部分获取块，即使是 OpenAI 强大的 LLM 也无法正确回答。

```python
chain_response = qa_chain("DHA Group 拥有的物业的可出租面积是多少？")
chain_response["result"]  # 正确答案应该是 13,500 平方英尺
```
```output
"我不知道。"
```
```python
chain_response["source_documents"]
```
```output
[Document(page_content='1.6 Rentable Area of the Premises.', metadata={'id': '5b39a1ae84d51682328dca1467be211f', 'name': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'structure': 'lim h1', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/docset:CatalystGroup/dg:chunk[6]/dg:chunk'}),
 Document(page_content='1.6 Rentable Area of the Premises.', metadata={'id': '5b39a1ae84d51682328dca1467be211f', 'name': 'Sample Commercial Leases/Shorebucks LLC_AZ.pdf', 'structure': 'lim h1', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/docset:MenloGroup/dg:chunk[6]/dg:chunk'}),
 Document(page_content='1.6 Rentable Area of the Premises.', metadata={'id': '5b39a1ae84d51682328dca1467be211f', 'name': 'Sample Commercial Leases/Shorebucks LLC_FL.pdf', 'structure': 'lim h1', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/docset:Florida-section/docset:Florida/docset:Shorebucks/dg:chunk[5]/dg:chunk'}),
 Document(page_content='1.6 Rentable Area of the Premises.', metadata={'id': '5b39a1ae84d51682328dca1467be211f', 'name': 'Sample Commercial Leases/Shorebucks LLC_TX.pdf', 'structure': 'lim h1', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/docset:LandmarkLlc/dg:chunk[6]/dg:chunk'})]
```

乍一看，答案似乎是合理的，但是它是不正确的。如果您仔细查看此答案的源块，您将看到文档的分块未能将房东姓名和可出租面积放在同一上下文中，并且产生了无关的块，因此答案是不正确的（应该是 **13,500 平方英尺**）。

在这里，Docugami 可以提供帮助。如果用户一直在使用 Docugami，块会用不同的技术进行注释，并创建附加元数据。稍后将添加更多技术方法。

具体来说，让我们要求 Docugami 在其输出中返回 XML 标签以及其他元数据：

```python
loader = DocugamiLoader(docset_id="zo954yqy53wp")
loader.include_xml_tags = (
    True  # 为了从 Docugami 知识图谱中获得额外的语义信息
)
chunks = loader.load()
print(chunks[0].metadata)
```
```output
{'xpath': '/docset:OFFICELEASE-section/dg:chunk', 'id': '47297e277e556f3ce8b570047304560b', 'name': 'Sample Commercial Leases/Shorebucks LLC_AZ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_AZ.pdf', 'structure': 'h1 h1 p', 'tag': 'chunk Lease', 'Lease Date': 'March  29th , 2019', 'Landlord': 'Menlo Group', 'Tenant': 'Shorebucks LLC', 'Premises Address': '1564  E Broadway Rd ,  Tempe ,  Arizona  85282', 'Term of Lease': '96  full calendar months', 'Square Feet': '16,159'}
```

我们可以使用 [自查询检索器](/docs/how_to/self_query) 来提高查询准确性，利用这些额外的元数据：

```python
!poetry run pip install --upgrade lark --quiet
```
```python
from langchain.chains.query_constructor.schema import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_community.vectorstores.chroma import Chroma
EXCLUDE_KEYS = ["id", "xpath", "structure"]
metadata_field_info = [
    AttributeInfo(
        name=key,
        description=f"这个片段的 {key}",
        type="string",
    )
    for key in chunks[0].metadata
    if key.lower() not in EXCLUDE_KEYS
]
document_content_description = "这个片段的内容"
llm = OpenAI(temperature=0)
vectordb = Chroma.from_documents(documents=chunks, embedding=embedding)
retriever = SelfQueryRetriever.from_llm(
    llm, vectordb, document_content_description, metadata_field_info, verbose=True
)
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=retriever,
    return_source_documents=True,
    verbose=True,
)
```

让我们再次运行相同的问题。它返回了正确的结果，因为所有的片段都有元数据键/值对，这些键/值对携带了关于文档的重要信息，即使这些信息在物理上与生成答案的源片段相距很远。

```python
qa_chain(
    "DHA Group 拥有的物业的可租面积是多少？"
)  # 正确答案应该是 13,500 平方英尺
```
```output
> 进入新的 RetrievalQA 链...
> 链结束。
```
```output
{'query': 'DHA Group 拥有的物业的可租面积是多少？',
 'result': ' DHA Group 拥有的物业的可租面积为 13,500 平方英尺。',
 'source_documents': [Document(page_content='1.6 Rentable Area of the Premises.', metadata={'Landlord': 'DHA Group', 'Lease Date': 'March  29th , 2019', 'Premises Address': '111  Bauer Dr ,  Oakland ,  New Jersey ,  07436', 'Square Feet': '13,500', 'Tenant': 'Shorebucks LLC', 'Term of Lease': '84  full calendar  months', 'id': '5b39a1ae84d51682328dca1467be211f', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'lim h1', 'tag': 'chunk', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/docset:DhaGroup/dg:chunk[6]/dg:chunk'}),
  Document(page_content='<RentableAreaofthePremises><SquareFeet>13,500 </SquareFeet>square feet. This square footage figure includes an add-on factor for Common Areas in the Building and has been agreed upon by the parties as final and correct and is not subject to challenge or dispute by either party. </RentableAreaofthePremises>', metadata={'Landlord': 'DHA Group', 'Lease Date': 'March  29th , 2019', 'Premises Address': '111  Bauer Dr ,  Oakland ,  New Jersey ,  07436', 'Square Feet': '13,500', 'Tenant': 'Shorebucks LLC', 'Term of Lease': '84  full calendar  months', 'id': '4c06903d087f5a83e486ee42cd702d31', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'div', 'tag': 'RentableAreaofthePremises', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/docset:DhaGroup/dg:chunk[6]/docset:RentableAreaofthePremises-section/docset:RentableAreaofthePremises'}),
  Document(page_content='<TheTermAnnualMarketRent>shall mean (i) for the initial Lease Year (“Year 1”) <Money>$2,239,748.00 </Money>per year (i.e., the product of the Rentable Area of the Premises multiplied by <Money>$82.00</Money>) (the “Year 1 Market Rent Hurdle”); (ii) for the Lease Year thereafter, <Percent>one hundred three percent (103%) </Percent>of the Year 1 Market Rent Hurdle, and (iii) for each Lease Year thereafter until the termination or expiration of this Lease, the Annual Market Rent Threshold shall be <AnnualMarketRentThreshold>one hundred three percent (103%) </AnnualMarketRentThreshold>of the Annual Market Rent Threshold for the immediately prior Lease Year. </TheTermAnnualMarketRent>', metadata={'Landlord': 'DHA Group', 'Lease Date': 'March  29th , 2019', 'Premises Address': '111  Bauer Dr ,  Oakland ,  New Jersey ,  07436', 'Square Feet': '13,500', 'Tenant': 'Shorebucks LLC', 'Term of Lease': '84  full calendar  months', 'id': '6b90beeadace5d4d12b25706fb48e631', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'div', 'tag': 'TheTermAnnualMarketRent', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/docset:GrossRentCredit-section/docset:GrossRentCredit/dg:chunk/dg:chunk/dg:chunk/dg:chunk[2]/docset:PercentageRent/dg:chunk[2]/dg:chunk[2]/docset:TenantSRevenue/dg:chunk[2]/docset:TenantSRevenue/dg:chunk[3]/docset:TheTermAnnualMarketRent-section/docset:TheTermAnnualMarketRent'}),
  Document(page_content='1.11 Percentage Rent.\n (a) <GrossRevenue><Percent>55% </Percent>of Gross Revenue to Landlord until Landlord receives Percentage Rent in an amount equal to the Annual Market Rent Hurdle (as escalated); and </GrossRevenue>', metadata={'Landlord': 'DHA Group', 'Lease Date': 'March  29th , 2019', 'Premises Address': '111  Bauer Dr ,  Oakland ,  New Jersey ,  07436', 'Square Feet': '13,500', 'Tenant': 'Shorebucks LLC', 'Term of Lease': '84  full calendar  months', 'id': 'c8bb9cbedf65a578d9db3f25f519dd3d', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'lim h1 lim p', 'tag': 'chunk GrossRevenue', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/docset:GrossRentCredit-section/docset:GrossRentCredit/dg:chunk/dg:chunk/dg:chunk/docset:PercentageRent/dg:chunk[1]/dg:chunk[1]'})]}
这次的答案是正确的，因为自查询检索器在元数据的房东属性上创建了一个过滤器，正确地过滤出了关于DHA集团房东的文件。结果中的源块都与该房东相关，即使在包含正确答案的特定块中没有直接提到该房东，这也提高了答案的准确性。
# 高级主题：使用文档知识图谱层次进行小到大的检索
文档本质上是半结构化的，DocugamiLoader能够在返回的块上导航文档的语义和结构轮廓，以提供父块的引用。这在与[MultiVector Retriever](/docs/how_to/multi_vector)进行[小到大的](https://www.youtube.com/watch?v=ihSiRrOUwmg)检索时非常有用。
要获取父块的引用，可以将`loader.parent_hierarchy_levels`设置为非零值。
```python
from typing import Dict, List
from docugami_langchain.document_loaders import DocugamiLoader
from langchain_core.documents import Document
loader = DocugamiLoader(docset_id="zo954yqy53wp")
loader.include_xml_tags = True  # 从Docugami知识图谱中获取额外的语义信息
loader.parent_hierarchy_levels = 3  # 扩展上下文
loader.max_text_length = 1024 * 8  # 8K个字符大约是2K个标记（参考：https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them）
loader.include_project_metadata_in_doc_metadata = False  # 不过滤向量元数据，以减轻向量的负担
chunks: List[Document] = loader.load()
# 构建父块和子块的分别映射
parents_by_id: Dict[str, Document] = {}
children_by_id: Dict[str, Document] = {}
for chunk in chunks:
    chunk_id = chunk.metadata.get("id")
    parent_chunk_id = chunk.metadata.get(loader.parent_id_key)
    if not parent_chunk_id:
        # 父块
        parents_by_id[chunk_id] = chunk
    else:
        # 子块
        children_by_id[chunk_id] = chunk
```
```python
# 探索一些父块关系
for id, chunk in list(children_by_id.items())[:5]:
    parent_chunk_id = chunk.metadata.get(loader.parent_id_key)
    if parent_chunk_id:
        # 子块有设置父块id
        print(f"PARENT CHUNK {parent_chunk_id}: {parents_by_id[parent_chunk_id]}")
        print(f"CHUNK {id}: {chunk}")
```
```output

PARENT CHUNK 7df09fbfc65bb8377054808aac2d16fd: page_content='OFFICE LEASE\n THIS OFFICE LEASE\n <Lease>(the "Lease") is made and entered into as of <LeaseDate>March 29th, 2019</LeaseDate>, by and between Landlord and Tenant. "Date of this Lease" shall mean the date on which the last one of the Landlord and Tenant has signed this Lease. </Lease>\nW I T N E S S E T H\n <TheTerms> Subject to and on the terms and conditions of this Lease, Landlord leases to Tenant and Tenant hires from Landlord the Premises. </TheTerms>\n1. BASIC LEASE INFORMATION AND DEFINED TERMS.\nThe key business terms of this Lease and the defined terms used in this Lease are as follows:' metadata={'xpath': '/docset:OFFICELEASE-section/dg:chunk', 'id': '7df09fbfc65bb8377054808aac2d16fd', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'h1 h1 p h1 p lim h1 p', 'tag': 'chunk Lease chunk TheTerms'}

CHUNK 47297e277e556f3ce8b570047304560b: page_content='OFFICE LEASE\n THIS OFFICE LEASE\n <Lease>(the "Lease") is made and entered into as of <LeaseDate>March 29th, 2019</LeaseDate>, by and between Landlord and Tenant. "Date of this Lease" shall mean the date on which the last one of the Landlord and Tenant has signed this Lease. </Lease>' metadata={'xpath': '/docset:OFFICELEASE-section/dg:chunk', 'id': '47297e277e556f3ce8b570047304560b', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'h1 h1 p', 'tag': 'chunk Lease', 'doc_id': '7df09fbfc65bb8377054808aac2d16fd'}

```
**写字楼租赁合同**
**本写字楼租赁合同**
《租赁协议》（以下简称“本协议”）于2018年1月8日签订，出租方和承租方为本协议的签署方。本协议的“签订日期”指出租方和承租方中最后一方签署本协议的日期。
**见证如下**
《条款》根据本协议的条款和条件，出租方将出租房屋给承租方，承租方从出租方租用该房屋。
1. **基本租赁信息和定义条款**
本租赁协议的关键业务条款和使用的定义条款如下：
1.1 **出租方**
出租方：Catalyst Group LLC
1.2 **承租方**
承租方：Shorebucks LLC
**基本租赁信息和定义条款**
**1.3 建筑物**
建筑物：位于Bellevue市Main Street 600号，邮编98004的房屋，该建筑物位于该项目内。[20]
**CHUNK a95971d693b7aa0f6640df1fbd18c2ba: page_content='本租赁协议的关键业务条款和本租赁协议中使用的定义条款如下：' metadata={'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/dg:chunk', 'id': 'a95971d693b7aa0f6640df1fbd18c2ba', 'name': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'structure': 'p', 'tag': 'chunk', 'doc_id': 'c942010baaf76aa4d4657769492f6edb'}
**父级 CHUNK f34b649cde7fc4ae156849a56d690495: page_content='见证\n <条款> 根据本租赁协议的条款和条件，房东将房屋出租给租户，租户从房东处租用房屋。 </条款>\n1. 基本租赁信息和定义条款。\n<基本租赁信息和定义条款>本租赁协议的关键业务条款和本租赁协议中使用的定义条款如下： </基本租赁信息和定义条款>\n1.1 房东。\n <房东><房东>Menlo Group</房东>，一家在<美国州>特拉华</美国州>注册的有限责任公司，有权在<美国州>亚利桑那</美国州>从事业务。 </房东>\n1.2 租户。\n <租户>Shorebucks LLC </租户>\n1.3 建筑物。\n <建筑物>位于<租赁地址><租赁街道地址><租赁>1564 </租赁><租赁>E Broadway Rd</租赁></租赁街道地址>，<城市>坦佩</城市>，<美国州>亚利桑那 </美国州><租赁>85282</租赁></租赁地址>的房屋。该建筑物位于该项目内。 </建筑物>\n1.4 项目。\n <项目>位于Shorebucks Office的土地和建筑物及改进物，地址为<ShorebucksOffice地址><ShorebucksOffice街道地址><ShorebucksOffice>6 </ShorebucksOffice><ShorebucksOffice6>位于<号码>1564 </号码>E Broadway Rd</ShorebucksOffice6></ShorebucksOffice街道地址>，<城市>坦佩</城市>，<美国州>亚利桑那 </美国州><号码>85282</号码></ShorebucksOffice地址>。该项目在本租赁协议的附件“A”中有法律描述。 </项目>' metadata={'xpath': '/dg:chunk/docset:WITNESSETH-section/dg:chunk', 'id': 'f34b649cde7fc4ae156849a56d690495', 'name': 'Sample Commercial Leases/Shorebucks LLC_AZ.docx', 'source': 'Sample Commercial Leases/Shorebucks LLC_AZ.docx', 'structure': 'h1 p lim h1 div lim h1 div lim h1 div lim h1 div lim h1 div', 'tag': 'chunk TheTerms BASICLEASEINFORMATIONANDDEFINEDTERMS chunk Landlord chunk Tenant chunk Building chunk Project'}
**CHUNK 21b4d9517f7ccdc0e3a028ce5043a2a0: page_content='1.1 房东。\n <房东><房东>Menlo Group</房东>，一家在<美国州>特拉华 </美国州>注册的有限责任公司，有权在<美国州>亚利桑那</美国州>从事业务。 </房东>' metadata={'xpath': '/dg:chunk/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk[1]/dg:chunk[1]/dg:chunk/dg:chunk[2]/dg:chunk', 'id': '21b4d9517f7ccdc0e3a028ce5043a2a0', 'name': 'Sample Commercial Leases/Shorebucks LLC_AZ.docx', 'source': 'Sample Commercial Leases/Shorebucks LLC_AZ.docx', 'structure': 'lim h1 div', 'tag': 'chunk Landlord', 'doc_id': 'f34b649cde7fc4ae156849a56d690495'}
```

**<标识>租户不得在项目的任何部分放置标识。但是，房东将允许租户在入口附近经房东批准的位置放置标有其名称的标识（由租户承担成本），并将在大厦的目录中提供其名称的单一列表（由房东承担成本），所有这些都要符合房东针对项目不时采纳的标准。目录中的任何更改或额外列表将按照当时的大厦标准收费提供（视空间是否可用而定）。</标识>

43090337ed2409e0da24ee07e2adbe94

**<外观>租户同意，所有从租赁物外部可见的标识、遮阳篷、防护门、安全设备和其他设施均须经房东事先书面批准，如有必要，还须经过纽约市<组织>地标保护委员会</组织>的事先批准，并且不得干扰或阻挡相邻商店的任何一家，但房东不得无理地拒绝租户希望安装的标识。租户同意，任何允许的标识、遮阳篷、防护门、安全设备和其他设施均应由租户自行承担成本和费用，专业准备并庄重地安装，并经房东事先书面批准，批准不得无理地被拒绝、延迟或附加条件，并须遵守房东不时可能施加的合理规定和限制。租户应向房东提交拟安装标识和其他设施的图纸，显示其尺寸、颜色、照明和一般外观，以及说明其将如何固定在租赁物上。除非房东书面批准，租户不得开始安装拟议的标识和其他设施。租户不得安装任何霓虹灯标识。前述标识仅用于标识租户的业务。未经事先获得房东的书面同意，不得对标识和其他设施进行任何更改，该同意不得无理地被拒绝、延迟或附加条件。租户应自行承担成本和费用，从任何和所有具有管辖权的城市、州和其他机构获得并向房东展示所需的许可证或批准证书，涵盖所述标识或其他设施的设置、安装、维护或使用，并且租户应保持所述标识和其他设施及其附属设施处于良好状态并得到房东的满意，并符合任何和所有具有管辖权的公共机构的命令、规定、要求和规则。房东同意租户所述的初始标识，详见附件D。</外观>

54ddfc3e47f41af7e747b2bc439ea96b

```python
# 查询检索器，应返回父级（使用MMR，因为这是上面设置的搜索类型）
retrieved_parent_docs = retriever.invoke(
    "Birch Street允许在其物业上放置哪些标识？"
)
for chunk in retrieved_parent_docs:
    print(chunk.page_content)
    print(chunk.metadata["id"])
```

22. 押金

<SECURITYDEPOSIT>押金将由房东作为租户履行本租约的保证，包括支付租金。租户将押金视为房东的担保利益。押金可能与房东的其他资金混合使用，房东对押金不承担任何利息支付责任。房东可以将押金用于弥补租户的任何违约行为。如果房东这样使用了押金，租户应在收到房东通知后的五天内将足够金额补充到原始押金数额。押金不应被视为租金的预付或租户违约的赔偿措施，也不得作为房东对租户提起的任何诉讼的抗辩理由。</SECURITYDEPOSIT>

23. 政府法规

<GOVERNMENTALREGULATIONS>租户应自行承担成本，及时遵守所有政府机构的法律、法规和条例，包括1990年通过的《美国残疾人法案》（"ADA"）及其修正案，以及影响项目的所有记录契约和限制，涉及租户、其业务行为以及对租赁场所的使用和占用，包括因租户特定使用（而非一般办公使用）租赁场所或租户进行的改建工作而需要对公共区域进行的工作。</GOVERNMENTALREGULATIONS>

24. 标识

<SIGNS>租户不得在项目的任何部分放置标识。但是，房东将批准租户在租赁场所入口附近的指定位置（由租户承担费用）放置标有其名称的标识，并将向租户提供建筑物目录的单一列表（由房东承担费用），所有这些都应符合房东针对项目制定的标准。目录中的任何更改或额外列表（视空间情况而定）应按照当时的建筑标准收费提供。</SIGNS>

25. 经纪人

<BROKER>房东和租户各自声明并保证，他们在与租赁场所有关的事务中未与任何经纪人或中介进行磋商或谈判，除了房东的经纪人和租户的经纪人。租户应对任何与本租约有关的房东经纪人和租户经纪人以外的任何房地产经纪人的佣金要求进行赔偿、辩护并使房东免受损害。房东应对与本租约有关的房东经纪人和租户经纪人应支付的任何租赁佣金以及任何与房东在本租约中进行交易的房地产经纪人的佣金要求进行赔偿、辩护并使租户免受损害。本条款的规定应在本租约的到期或提前终止后继续有效。</BROKER>

26. 租期结束

<ENDOFTERM>在本租约到期或提前终止时，租户应将租赁场所以良好的状态交还给房东，除了合理的磨损和损耗。房东或租户对租赁场所进行的所有改建将在租约期满或提前终止时成为房东的财产。在租约期满或提前终止时，租户应自行承担费用，将租赁场所内的所有个人财产、计算机和电信布线以及房东通知租户指定的所有改建工程移除。租户还应修复因此造成的任何损坏。在租约期满或提前终止后，租户的任何财产项目如若留在租赁场所内，房东可以选择将其视为废弃物，且在不通知租户或其他任何方的情况下，将其作为自己的财产保留并处置，处置方式由房东决定，费用由租户承担。</ENDOFTERM>

27. 律师费

<律师费> 除非本租约另有规定，任何因本租约引起的或与本租约有关的诉讼或其他争议解决程序，包括仲裁，无论是基于侵权行为还是要求禁令、宣告或临时救济的诉讼，胜诉方均有权从败诉方那里获得实际律师费和费用，包括诉讼费用、确定或根据本条款所欠费用或费用金额的诉讼费用，以及与破产、上诉或追讨程序有关的费用。除房东或租户外，任何个人或实体均无权根据本段获得费用。此外，如果房东成为影响房屋或涉及本租约或租户根据本租约的利益的任何诉讼或程序的一方，除了房东和租户之间的诉讼外，或者如果房东聘请律师收取本租约项下的任何金额，或者强制执行本租约的任何协议、条件、契约、规定或约定，而无需提起诉讼，那么由房东发生的成本、费用以及合理的律师费和支出应由租户支付。 </律师费>

![图片描述](https://pic.com/43090337ed2409e0da24ee07e2adbe94)

<租户独自承担成本> 租户应自行承担所有从房屋中清除和处理所有垃圾、废物和废料的费用。租户应使所有垃圾、废物和废料存放在房屋内，直到关门前的三十（30）分钟。但租户可以在上述句子规定的时间之后，根据法律允许的范围，在关门前的次日6:00 A.M.之前将垃圾放在房屋外等待清理。垃圾应放置在房屋前人行道的边缘，距建筑物主入口最远的位置，或者房东指定的建筑物前的其他位置。 </租户独自承担成本>

<其独自承担成本> 租户应自行承担所有费用，同意按照最佳现行方法，采取一切合理努力预防和消灭害虫、老鼠、霉菌、真菌、过敏原、细菌及房屋内存在的所有其他类似情况。租户应自行承担费用，定期对房屋进行灭虫，以使房东合理满意，并雇用持牌的灭虫公司。房东不负责房屋的任何清洁、垃圾清理、管家或类似服务，如果房屋内存在本条款所述的任何情况，租户不得要求房东减免、抵消或信用。 </其独自承担成本>

42B. 人行道使用和维护

<人行道> 租户应自行承担费用，保持房屋前人行道离路缘向街道内清洁，无垃圾、废物、废料、多余水、积雪和冰，租户应支付任何罚款、费用或因未能做到而产生的额外租金。如果租户经营人行道咖啡馆，租户应自行承担费用，根据需要维护、修理并更换房屋前的人行道和房屋地下室的金属活动门。租户应在任何侧门使用时在所有侧面张贴警告标志和锥形标志，并在任何侧门开启时始终横跨安全杆。 </人行道>

<展示> 在任何情况下，租户不得使用或允许使用房屋外部的空间进行展示、销售或任何其他类似活动；除非[1]在合法持牌的“街头市集”类型计划下，或者[<Number>2</Number>]如果当地的区域规划、社区委员会[如适用]和其他市政法律、规则和法规允许使用人行道咖啡馆，如果是这种情况，该运营应严格遵守所有上述要求和条件。在任何情况下，租户不得使用或允许使用任何广告媒介和/或扩音器和/或声音放大器和/或广播电台或电视广播，这些可能在房屋外被听到，或者不符合房东当时有效的合理规则和规定。 </展示>

42C. 店面维护

## 外墙和安全门

租户同意按照房东的合理要求，每月或更频繁地清洗店面，包括外墙和安全门，从顶部到地面，并根据房东的判断，对店面内外的所有窗户、玻璃板和其他玻璃以及安全门进行必要的维修和更换。如果租户未能按照本条款维护店面，房东可以自行承担费用，并将费用作为额外租金计入租户账单中。

42D. 音乐、噪音和振动

4474c92ae7ccec9184ed2fef9f072734

```