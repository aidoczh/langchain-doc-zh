# PubMed

[PubMed®](https://pubmed.ncbi.nlm.nih.gov/) 由美国国家生物技术信息中心、国家医学图书馆发布，收录了来自 MEDLINE、生命科学期刊和在线图书的超过 3500 万篇生物医学文献引用。引用可能包括指向 PubMed Central 和出版商网站的全文内容的链接。

本笔记介绍了如何使用 PubMed 作为检索器

```python
from langchain_community.retrievers import PubMedRetriever
```

```python
retriever = PubMedRetriever()
```

```python
retriever.invoke("chatgpt")
```

```output
[Document(page_content='', metadata={'uid': '37549050', 'Title': 'ChatGPT: "To Be or Not to Be" in Bikini Bottom.', 'Published': '--', 'Copyright Information': ''}),
 Document(page_content="BACKGROUND: ChatGPT is a large language model that has performed well on professional examinations in the fields of medicine, law, and business. However, it is unclear how ChatGPT would perform on an examination assessing professionalism and situational judgement for doctors.\nOBJECTIVE: We evaluated the performance of ChatGPT on the Situational Judgement Test (SJT): a national examination taken by all final-year medical students in the United Kingdom. This examination is designed to assess attributes such as communication, teamwork, patient safety, prioritization skills, professionalism, and ethics.\nMETHODS: All questions from the UK Foundation Programme Office's (UKFPO's) 2023 SJT practice examination were inputted into ChatGPT. For each question, ChatGPT's answers and rationales were recorded and assessed on the basis of the official UK Foundation Programme Office scoring template. Questions were categorized into domains of Good Medical Practice on the basis of the domains referenced in the rationales provided in the scoring sheet. Questions without clear domain links were screened by reviewers and assigned one or multiple domains. ChatGPT's overall performance, as well as its performance across the domains of Good Medical Practice, was evaluated.\nRESULTS: Overall, ChatGPT performed well, scoring 76% on the SJT but scoring full marks on only a few questions (9%), which may reflect possible flaws in ChatGPT's situational judgement or inconsistencies in the reasoning across questions (or both) in the examination itself. ChatGPT demonstrated consistent performance across the 4 outlined domains in Good Medical Practice for doctors.\nCONCLUSIONS: Further research is needed to understand the potential applications of large language models, such as ChatGPT, in medical education for standardizing questions and providing consistent rationales for examinations assessing professionalism and ethics.", metadata={'uid': '37548997', 'Title': 'Performance of ChatGPT on the Situational Judgement Test-A Professional Dilemmas-Based Examination for Doctors in the United Kingdom.', 'Published': '2023-08-07', 'Copyright Information': '©Robin J Borchert, Charlotte R Hickman, Jack Pepys, Timothy J Sadler. Originally published in JMIR Medical Education (https://mededu.jmir.org), 07.08.2023.'}),
 Document(page_content='', metadata={'uid': '37548971', 'Title': "Large Language Models Answer Medical Questions Accurately, but Can't Match Clinicians' Knowledge.", 'Published': '2023-08-07', 'Copyright Information': ''})]
```