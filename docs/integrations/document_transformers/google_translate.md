# 谷歌翻译

[谷歌翻译](https://translate.google.com/) 是由谷歌开发的多语言神经机器翻译服务，可将文本、文件和网站从一种语言翻译成另一种语言。

`GoogleTranslateTransformer` 允许您使用[谷歌云翻译 API](https://cloud.google.com/translate)来翻译文本和 HTML。

要使用它，您应该安装了 `google-cloud-translate` Python 包，并且有一个启用了[翻译 API](https://cloud.google.com/translate/docs/setup)的谷歌云项目。此转换器使用[高级版 (v3)](https://cloud.google.com/translate/docs/intro-to-v3)。

- [谷歌神经机器翻译](https://en.wikipedia.org/wiki/Google_Neural_Machine_Translation)

- [一个用于机器翻译的神经网络，达到了生产规模](https://blog.research.google/2016/09/a-neural-network-for-machine.html)

```python
%pip install --upgrade --quiet  google-cloud-translate
```

```python
from langchain_core.documents import Document
from langchain_google_community import GoogleTranslateTransformer
```

## 输入

这是我们将要翻译的文档

```python
sample_text = """[Generated with Google Bard]
Subject: Key Business Process Updates
Date: Friday, 27 October 2023
Dear team,
I am writing to provide an update on some of our key business processes.
Sales process
We have recently implemented a new sales process that is designed to help us close more deals and grow our revenue. The new process includes a more rigorous qualification process, a more streamlined proposal process, and a more effective customer relationship management (CRM) system.
Marketing process
We have also revamped our marketing process to focus on creating more targeted and engaging content. We are also using more social media and paid advertising to reach a wider audience.
Customer service process
We have also made some improvements to our customer service process. We have implemented a new customer support system that makes it easier for customers to get help with their problems. We have also hired more customer support representatives to reduce wait times.
Overall, we are very pleased with the progress we have made on improving our key business processes. We believe that these changes will help us to achieve our goals of growing our business and providing our customers with the best possible experience.
If you have any questions or feedback about any of these changes, please feel free to contact me directly.
Thank you,
Lewis Cymbal
CEO, Cymbal Bank
"""
```

在初始化 `GoogleTranslateTransformer` 时，您可以包含以下参数来配置请求。

- `project_id`：谷歌云项目 ID。

- `location`：（可选）翻译模型位置。

  - 默认值：`global`

- `model_id`：（可选）要使用的翻译[模型 ID][models]。

- `glossary_id`：（可选）要使用的[词汇表 ID][glossaries]。

- `api_endpoint`：（可选）要使用的[区域端点][endpoints]。

[models]: https://cloud.google.com/translate/docs/advanced/translating-text-v3#comparing-models

[glossaries]: https://cloud.google.com/translate/docs/advanced/glossary

[endpoints]: https://cloud.google.com/translate/docs/advanced/endpoints

```python
documents = [Document(page_content=sample_text)]
translator = GoogleTranslateTransformer(project_id="<YOUR_PROJECT_ID>")
```

## 输出

翻译文档后，结果将作为一个新文档返回，其中 `page_content` 被翻译成目标语言。

您可以为 `transform_documents()` 方法提供以下关键字参数：

- `target_language_code`：输出文档的[ISO 639][iso-639]语言代码。

  - 支持的语言，请参阅[语言支持][supported-languages]。

- `source_language_code`：（可选）输入文档的[ISO 639][iso-639]语言代码。

  - 如果未提供，将自动检测语言。

- `mime_type`：（可选）输入文本的[媒体类型][media-type]。

  - 选项：`text/plain`（默认），`text/html`。

[iso-639]: https://en.wikipedia.org/wiki/ISO_639

[supported-languages]: https://cloud.google.com/translate/docs/languages

[media-type]: https://en.wikipedia.org/wiki/Media_type

```python
translated_documents = translator.transform_documents(
    documents, target_language_code="es"
)
```

```python
for doc in translated_documents:
    print(doc.metadata)
    print(doc.page_content)
```

```output
{'model': '', 'detected_language_code': 'en'}
[生成自 Google Bard]
主题：关键业务流程更新
日期：2023年10月27日，星期五
亲爱的团队，
我写信是为了向您提供一些关键业务流程的更新。
销售流程
我们最近实施了一个新的销售流程，旨在帮助我们完成更多交易并增加收入。新流程包括更严格的资格认证流程、更简化的提案流程和更有效的客户关系管理（CRM）系统。
营销流程
我们还改进了我们的营销流程，重点是创建更具针对性和吸引力的内容。我们还使用更多的社交媒体和付费广告来触达更广泛的受众。
客户服务流程
我们还对我们的客户服务流程进行了一些改进。我们实施了一个新的客户支持系统，使客户更容易获得帮助解决问题。我们还雇佣了更多的客户支持代表，以减少等待时间。
总体而言，我们对改进关键业务流程取得的进展非常满意。我们相信这些变化将帮助我们实现增长业务和为客户提供最佳体验的目标。
如果您对这些变化有任何问题或反馈，请随时直接与我联系。
谢谢，
Lewis Cymbal
首席执行官，Cymbal Bank
```