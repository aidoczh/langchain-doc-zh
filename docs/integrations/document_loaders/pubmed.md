# PubMed

>[PubMed®](https://pubmed.ncbi.nlm.nih.gov/)由`美国国家生物技术信息中心、国家医学图书馆`提供，包括来自`MEDLINE`、生命科学期刊和在线图书的超过3500万条生物医学文献引用。引用可能包括来自`PubMed Central`和出版商网站的全文内容链接。

```python
from langchain_community.document_loaders import PubMedLoader
```

```python
loader = PubMedLoader("chatgpt")
```

```python
docs = loader.load()
```

```python
len(docs)
```

```output
3
```

```python
docs[1].metadata
```

```output
{'uid': '37548997',
 'Title': 'Performance of ChatGPT on the Situational Judgement Test-A Professional Dilemmas-Based Examination for Doctors in the United Kingdom.',
 'Published': '2023-08-07',
 'Copyright Information': '©Robin J Borchert, Charlotte R Hickman, Jack Pepys, Timothy J Sadler. Originally published in JMIR Medical Education (https://mededu.jmir.org), 07.08.2023.'}
```

```python
docs[1].page_content
```

```output
"背景：ChatGPT是一个大型语言模型，在医学、法律和商业领域的专业考试中表现良好。然而，目前尚不清楚ChatGPT在评估医生的专业素养和情境判断的考试中的表现如何。
目标：我们评估了ChatGPT在情境判断测试（SJT）上的表现：这是英国所有最后一年医学生参加的国家考试。该考试旨在评估沟通、团队合作、患者安全、优先级确定技能、专业素养和伦理等属性。
方法：将英国基金会项目办公室（UKFPO）2023年SJT模拟考试的所有问题输入ChatGPT。针对每个问题，记录ChatGPT的答案和理由，并根据官方的UK基金会项目办公室评分模板进行评估。根据评分表中提供的理由所涉及的领域，将问题分类为《医德良好实践》的领域。没有明确领域链接的问题经过审阅人员筛选，并分配一个或多个领域。评估了ChatGPT的整体表现，以及其在医生《医德良好实践》领域的表现。
结果：总体而言，ChatGPT表现良好，在SJT上得分为76%，但只有少数问题（9%）得满分，这可能反映了ChatGPT在情境判断上的缺陷或者考试本身在问题推理方面的不一致性（或两者兼有）。ChatGPT在医生《医德良好实践》领域表现一致。
结论：需要进一步研究了解大型语言模型（如ChatGPT）在医学教育中的潜在应用，以标准化问题并为评估专业素养和伦理的考试提供一致的理由。"
```