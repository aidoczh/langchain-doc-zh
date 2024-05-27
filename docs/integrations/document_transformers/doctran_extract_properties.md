

# Doctran: 提取属性

我们可以使用 [Doctran](https://github.com/psychic-api/doctran) 库提取文档的有用特征，该库利用 OpenAI 的函数调用功能来提取特定的元数据。

从文档中提取元数据对于各种任务都很有帮助，包括：

* **分类：** 将文档分类到不同的类别中

* **数据挖掘：** 提取可用于数据分析的结构化数据

* **风格转换：** 更改文本的写作方式，使其更接近预期的用户输入，从而改善向量搜索结果

```python
%pip install --upgrade --quiet  doctran
```

```python
import json
from langchain_community.document_transformers import DoctranPropertyExtractor
from langchain_core.documents import Document
```

```python
from dotenv import load_dotenv
load_dotenv()
```

```output
True
```

## 输入

这是我们将从中提取属性的文档。

```python
sample_text = """[Generated with ChatGPT]
Confidential Document - For Internal Use Only
Date: July 1, 2023
Subject: Updates and Discussions on Various Topics
Dear Team,
I hope this email finds you well. In this document, I would like to provide you with some important updates and discuss various topics that require our attention. Please treat the information contained herein as highly confidential.
Security and Privacy Measures
As part of our ongoing commitment to ensure the security and privacy of our customers' data, we have implemented robust measures across all our systems. We would like to commend John Doe (email: john.doe@example.com) from the IT department for his diligent work in enhancing our network security. Moving forward, we kindly remind everyone to strictly adhere to our data protection policies and guidelines. Additionally, if you come across any potential security risks or incidents, please report them immediately to our dedicated team at security@example.com.
HR Updates and Employee Benefits
Recently, we welcomed several new team members who have made significant contributions to their respective departments. I would like to recognize Jane Smith (SSN: 049-45-5928) for her outstanding performance in customer service. Jane has consistently received positive feedback from our clients. Furthermore, please remember that the open enrollment period for our employee benefits program is fast approaching. Should you have any questions or require assistance, please contact our HR representative, Michael Johnson (phone: 418-492-3850, email: michael.johnson@example.com).
Marketing Initiatives and Campaigns
Our marketing team has been actively working on developing new strategies to increase brand awareness and drive customer engagement. We would like to thank Sarah Thompson (phone: 415-555-1234) for her exceptional efforts in managing our social media platforms. Sarah has successfully increased our follower base by 20% in the past month alone. Moreover, please mark your calendars for the upcoming product launch event on July 15th. We encourage all team members to attend and support this exciting milestone for our company.
Research and Development Projects
In our pursuit of innovation, our research and development department has been working tirelessly on various projects. I would like to acknowledge the exceptional work of David Rodriguez (email: david.rodriguez@example.com) in his role as project lead. David's contributions to the development of our cutting-edge technology have been instrumental. Furthermore, we would like to remind everyone to share their ideas and suggestions for potential new projects during our monthly R&D brainstorming session, scheduled for July 10th.
Please treat the information in this document with utmost confidentiality and ensure that it is not shared with unauthorized individuals. If you have any questions or concerns regarding the topics discussed, please do not hesitate to reach out to me directly.
Thank you for your attention, and let's continue to work together to achieve our goals.
Best regards,
Jason Fan
Cofounder & CEO
Psychic
jason@psychic.dev
"""
print(sample_text)
```

```python
documents = [Document(page_content=sample_text)]
properties = [
    {
        "name": "category",
        "description": "邮件类型。",
        "type": "string",
        "enum": ["update", "action_item", "customer_feedback", "announcement", "other"],
        "required": True,
    },
    {
        "name": "mentions",
        "description": "邮件中提到的所有人的列表。",
        "type": "array",
        "items": {
            "name": "full_name",
            "description": "被提到的人的全名。",
            "type": "string",
        },
        "required": True,
    },
    {
        "name": "eli5",
        "description": "用儿童语言解释这封邮件给我听。",
        "type": "string",
        "required": True,
    },
]
property_extractor = DoctranPropertyExtractor(properties=properties)
```

## 输出

从文档中提取属性后，结果将作为包含在元数据中的新文档返回

```python
extracted_document = property_extractor.transform_documents(
    documents, properties=properties
)
```

```python
print(json.dumps(extracted_document[0].metadata, indent=2))
```

```output
{
  "extracted_properties": {
    "category": "update",
    "mentions": [
      "John Doe",
      "Jane Smith",
      "Michael Johnson",
      "Sarah Thompson",
      "David Rodriguez"
    ],
    "eli5": "这封邮件提供了关于各种主题的重要更新和讨论。它提到了安全和隐私措施的实施、人力资源更新和员工福利、营销活动和宣传活动，以及研发项目。它表彰了 John Doe、Jane Smith、Michael Johnson、Sarah Thompson 和 David Rodriguez 的贡献。它还提醒所有人遵守数据保护政策，参加员工福利计划，参加即将举行的产品发布活动，并在研发头脑风暴会议期间分享新项目的想法。"
  }
}
```