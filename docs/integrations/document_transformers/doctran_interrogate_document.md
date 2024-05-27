# Doctran：审问文档

在向量存储知识库中使用的文档通常以叙述或对话形式存储。然而，大多数用户查询以问题形式提出。如果我们在将文档向量化之前**将文档转换为问答格式**，我们可以增加检索相关文档的可能性，减少检索无关文档的可能性。

我们可以使用[Doctran](https://github.com/psychic-api/doctran)库来实现这一目标，该库使用OpenAI的函数调用功能来“审问”文档。

请查看[此笔记本](https://github.com/psychic-api/doctran/blob/main/benchmark.ipynb)，了解基于原始文档与审问文档的各种查询的向量相似度分数的基准测试。

```python
%pip install --upgrade --quiet  doctran
```

```python
import json
from langchain_community.document_transformers import DoctranQATransformer
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

这是我们将要审问的文档

```python
sample_text = """[Generated with ChatGPT]
机密文件 - 仅供内部使用
日期：2023年7月1日
主题：各种话题的更新和讨论
亲爱的团队，
希望这封邮件能找到你们一切安好。在这份文件中，我想向你们提供一些重要的更新，并讨论需要我们关注的各种话题。请将此处包含的信息视为高度机密。
安全和隐私措施
作为我们不断致力于确保客户数据安全和隐私的一部分，我们已在所有系统中实施了强有力的措施。我们要赞扬IT部门的John Doe（电子邮件：john.doe@example.com）在增强我们网络安全方面的勤奋工作。未来，我们提醒每个人严格遵守我们的数据保护政策和准则。此外，如果您发现任何潜在的安全风险或事件，请立即向我们专门的团队报告，联系邮箱为security@example.com。
人力资源更新和员工福利
最近，我们迎来了几位为各自部门做出重大贡献的新团队成员。我要表扬Jane Smith（社保号：049-45-5928）在客户服务方面的出色表现。Jane一直受到客户的积极反馈。此外，请记住我们的员工福利计划的开放报名期即将到来。如果您有任何问题或需要帮助，请联系我们的人力资源代表Michael Johnson（电话：418-492-3850，电子邮件：michael.johnson@example.com）。
营销倡议和活动
我们的营销团队一直在积极制定新策略，以提高品牌知名度并推动客户参与。我们要感谢Sarah Thompson（电话：415-555-1234）在管理我们的社交媒体平台方面的杰出努力。Sarah在过去一个月内成功将我们的关注者基数增加了20%。此外，请记住7月15日即将举行的产品发布活动。我们鼓励所有团队成员参加并支持我们公司的这一重要里程碑。
研发项目
在追求创新的过程中，我们的研发部门一直在为各种项目不懈努力。我要赞扬David Rodriguez（电子邮件：david.rodriguez@example.com）在项目负责人角色中的杰出工作。David对我们尖端技术的发展做出了重要贡献。此外，我们希望每个人在7月10日定期举行的研发头脑风暴会议上分享他们的想法和建议，以开展潜在的新项目。
请将此文档中的信息视为最机密，并确保不与未经授权的人员分享。如果您对讨论的话题有任何疑问或顾虑，请随时直接联系我。
感谢您的关注，让我们继续共同努力实现我们的目标。
此致，
Jason Fan
联合创始人兼首席执行官
Psychic
jason@psychic.dev
"""
print(sample_text)
```

```python
documents = [Document(page_content=sample_text)]
qa_transformer = DoctranQATransformer()
transformed_document = qa_transformer.transform_documents(documents)
```

## 输出

在对文档进行询问后，结果将作为一个新的文档返回，其中包含在元数据中提供的问题和答案。

```python
transformed_document = qa_transformer.transform_documents(documents)
print(json.dumps(transformed_document[0].metadata, indent=2))
```

```output
{
  "questions_and_answers": [
    {
      "question": "这个文档的目的是什么？",
      "answer": "这个文档的目的是提供重要的更新并讨论需要团队关注的各种话题。"
    },
    {
      "question": "如果有人遇到潜在的安全风险或事件应该怎么办？",
      "answer": "如果有人遇到潜在的安全风险或事件，应立即向 security@example.com 这个专门团队报告。"
    },
    {
      "question": "谁因增强网络安全而受到表扬？",
      "answer": "IT 部门的 John Doe 因增强网络安全而受到表扬。"
    },
    {
      "question": "需要协助员工福利的人应该联系谁？",
      "answer": "需要协助员工福利的人应该联系 HR 代表 Michael Johnson。他的电话号码是 418-492-3850，电子邮件是 michael.johnson@example.com。"
    },
    {
      "question": "哪些新团队成员在各自的部门做出了重要贡献？",
      "answer": "几位新团队成员在各自的部门做出了重要贡献。"
    },
    {
      "question": "谁因在客户服务方面的出色表现而受到认可？",
      "answer": "Jane Smith 因在客户服务方面的出色表现而受到认可。"
    },
    {
      "question": "谁成功地增加了社交媒体上的关注者数量？",
      "answer": "Sarah Thompson 成功地增加了社交媒体上的关注者数量。"
    },
    {
      "question": "即将举行的产品发布活动是什么时候？",
      "answer": "即将举行的产品发布活动将在7月15日举行。"
    },
    {
      "question": "谁因作为项目负责人的卓越工作而受到赞扬？",
      "answer": "David Rodriguez 因作为项目负责人的卓越工作而受到赞扬。"
    },
    {
      "question": "每月的研发头脑风暴会议安排在什么时候？",
      "answer": "每月的研发头脑风暴会议安排在7月10日。"
    }
  ]
}
```
